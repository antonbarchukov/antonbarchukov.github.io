<template>
    <div id="app">
        <h1>COVID-19 United States <sub>(General Overview)</sub></h1>
        <h4>Data as of {{ dateToday }}</h4>
        <div class="overview">
            <div class="d-flex flex-wrap justify-content-center">
                <div class="m-2">Active Cases: {{ Number(dataAPI.active).toLocaleString() }}</div>
                <div class="m-2">Total Cases: {{ Number(dataAPI.cases).toLocaleString() }}</div>
                <div class="m-2">Deaths: {{ Number(dataAPI.deaths).toLocaleString() }}</div>
                <div class="m-2">Critical: {{ Number(dataAPI.critical).toLocaleString() }}</div>
                <div class="m-2">Recovered: {{ Number(dataAPI.recovered).toLocaleString() }}</div>
                <div class="m-2">Total Tests: {{ Number(dataAPI.totalTests).toLocaleString() }}</div>
            </div>
            <div class="d-flex justify-content-center">
                <div class="m-2 font-weight-bold">New Cases: {{ Number(dataAPI.todayCases).toLocaleString() }}</div>
                <div class="m-2 font-weight-bold">New Deaths: {{ Number(dataAPI.todayDeaths).toLocaleString() }}</div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Overview',
  data() {
    return {
        dataAPI: [],
        dateToday: null,
    };
  },
  mounted: function() {
    this.dateToday = this.getCurrentDate();
    this.getData();
  },
  methods: {
    getData() {
        axios.get('https://coronavirus-19-api.herokuapp.com/countries/usa')
        .then((response) => {
            this.dataAPI = response.data;
        })
        .catch((error) => {
            console.log(error);
        });
    }, 
    getCurrentDate() {
        var myDate = new Date();
        var month = ('0' + (myDate.getMonth() + 1)).slice(-2);
        var date = ('0' + myDate.getDate()).slice(-2);
        var year = myDate.getFullYear();
        return year + '-' + month + '-' + date;
    }
  },

}
</script>
<style scoped>
    .overview {
        font-size: 1.1rem;
    }
</style>