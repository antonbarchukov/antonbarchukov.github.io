<template>
    <div id="app">
        <h1>COVID-19 United States <sub>(State Chart)</sub></h1>
        <h4>Data as of {{ dateToday }}</h4>
        <div class="table-responsive overflow-auto">
            <table v-if="dataAPILive" class="table table-striped">
            <thead class="thead-dark sticky-top ">
                <tr>
                    <th>State</th>
                    <th>Total Cases</th>
                    <th>New Cases</th>
                    <th>Total Deaths</th>
                    <th>New Deaths</th>
                    <th>Active Cases</th>
                    <th>Total Cases/ 1M</th>
                    <th>Deaths/ 1M</th>
                    <th>Total Tests</th>
                </tr>
            </thead>
            <tbody>
                <template v-for="data in dataAPILive.table">
                    <tr :key="data.USAState" data-toggle="modal" data-target="#dataChart" :data-province="data.USAState" v-on:click="getModal(data.USAState)">
                        <td>{{ data.USAState }}</td>
                        <td>{{ data.TotalCases }}</td>
                        <td>{{ data.NewCases }}</td>
                        <td>{{ data.TotalDeaths }}</td>
                        <td>{{ data.NewDeaths }}</td>
                        <td>{{ data.ActiveCases }}</td>
                        <td>{{ data.Tot_Cases_1M_Pop }}</td>
                        <td>{{ data.Deaths_1M_Pop }}</td>
                        <td>{{ data.TotalTests }}</td>
                    </tr>

                    <div class="modal fade" id="dataChart" tabindex="-1" role="dialog" aria-hidden="true" :key="data.TotalCases">
                        <div class="modal-dialog modal-dialog-centered" role="document">
                            <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title"></h5>
                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                                </button>
                            </div>
                            <div class="modal-body">
                                <highcharts :options="chartOptions"></highcharts>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            </div>
                            </div>
                        </div>
                    </div>
                </template>
            </tbody>
            </table>
        </div>
    </div>
</template>

<script>
import $ from "jquery";
import axios from "axios";

export default {
  name: "Chart",
  components: {},
  data() {
    return {
      dataAPIs: [],
      dataAPILive: null,
      dateSafeAPI: null,
      dateToday: null,
      historyNumber: 14,
      chartOptions: {
        series: [
          {
            data: []
          }
        ],
        xAxis: {
          type: "datetime"
        },
        legend: {
          enabled: false
        },
        title: {
          text: "Active Case History"
        },
        tooltip: {
          formatter: function() {
            return this.y;
          }
        }
      }
    };
  },
  mounted: function() {
    this.dateSafeAPI = this.getYesterdayDate(); //safe date for api's to be updated to
    this.dateToday = this.getCurrentDate(); //get data for today
    this.fetchHistory(); //fetches latest data (historyNumber of days)
  },
  methods: {
    getCurrentDate() {
      var myDate = new Date();
      var month = ("0" + (myDate.getMonth() + 1)).slice(-2);
      var date = ("0" + myDate.getDate()).slice(-2);
      var year = myDate.getFullYear();
      return year + "-" + month + "-" + date;
    },
    getYesterdayDate() {
      var myDate = new Date();
      var month = ("0" + (myDate.getMonth() + 1)).slice(-2);
      var date = ("0" + myDate.getDate()).slice(-2);
      var year = myDate.getFullYear();
      return year + "-" + month + "-" + (parseInt(date) - 1).toString();
    },
    //gets the right modal prepared to show after click on each state (sets the right highcharts data and shows the modal with right title)
    getModal(province) {
      this.getHistory(province);
      $("#dataChart").on("show.bs.modal", function(event) {
        var button = $(event.relatedTarget);
        var selectedProvince = button.data("province");
        var modal = $(this);
        modal.find(".modal-title").text(selectedProvince);
      });
    },
    //fetches history for the past historyNumber days(if available) days, plus pulls in 2 different sources for the information, 1 safe(1 day delayed displayed in charts), 1 current information (live displayed in table).
    fetchHistory() {
      //axios request for historyNumber days of safe info
      let requests = [];
      for (let i = 0; i < this.historyNumber; i++) {
        let aDate = new Date(this.dateSafeAPI);
        aDate.setDate(aDate.getDate() - i);
        let sDate = aDate.toISOString().split("T")[0];
        requests.push(
          axios.get(
            "https://covid-19-data.p.rapidapi.com/report/country/name?date-format=YYYY-MM-DD&format=json&date=" +
              sDate +
              "&name=USA",
            {
              headers: {
                "x-rapidapi-host": "covid-19-data.p.rapidapi.com",
                "x-rapidapi-key":
                  "cf374794a9msh7b1317c59c012f9p1825bcjsnd915fd72cd51"
              }
            }
          )
        );
      }
      axios //axios requests for historyNumber days
        .all(requests)
        .then(
          axios.spread((...responses) => {
            responses.map(obj => {
              this.dataAPIs.push(obj.data[0]);
            });
          })
        )
        .catch(errors => {
          // react on errors.
          console.log(errors);
        });

      //axios request for the last aval data (live)
      axios
        .get("https://covid19api.io/api/v1/CasesInAllUSStates")
        .then(response => {
          response.data.data[0].table.shift();
          response.data.data[0].table.pop();
          this.dataAPILive = response.data.data[0];
        });
    },
    getActiveCases(province) { //returns the active cases (latest data) for each province
      for(let data in this.dataAPILive.table) {
        if(this.dataAPILive.table[data].USAState == province){
          return Number(this.dataAPILive.table[data].ActiveCases.replace(/[^0-9.-]+/g,""));
        }
      }
    },
    getHistory(province) { //sets the highcharts info for last historyNumber days + latest data
      //sorts through the dataAPIs to find the right province and pulls last historyNumber days entries.
      let results = [];
      for (let i = 0; i < this.historyNumber; i++) {
        let history = this.dataAPIs[i].provinces;
        let date = this.dataAPIs[i].date;
        let filtered = history.filter(obj => { //we equal filtered when the for loop finds the right province, and it has active data.
          if (
            obj.province == province &&
            obj.active &&
            typeof obj.active != "undefined"
          ) {
            return obj.active;
          }
        });
        //if the filtered exists, we need to add it to the results to add to the highcharts data for display
        if (filtered[0] && filtered[0].active) {
          results.push({
            x: new Date(date).getTime(),
            y: filtered[0].active
          });
        }
      }
      //puts latest data in front of array
      results.unshift({
        x: new Date(this.getCurrentDate()).getTime(),
        y: this.getActiveCases(province)
      });
      //reverses array to match highcharts requirements
      this.chartOptions.series[0].data = results.reverse();
    }
  }
};
</script>

<style scoped>
table {
  border-collapse: collapse;
}

table,
th,
td {
  border: 1px solid black;
}

td:nth-child(3) {
  background-color: #ffbcb8;
}

td:nth-child(5) {
  background-color: #ffbcb8;
}

tr:hover {
  background-color: #d6d6d6 !important;
}
</style>
