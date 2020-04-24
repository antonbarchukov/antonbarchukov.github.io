<template>
  <div id="app">
    <Overview></Overview>
    <Chart></Chart>
  </div>
</template>

<script>
import Chart from "./components/Chart.vue"
import Overview from "./components/Overview.vue"

export default {
  name: 'App',
  components: {
    Chart,
    Overview
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
  font-size: 0.9rem;
  padding-left: 1rem;
  padding-right: 1rem;
  background-color: #f0f6ff;
}

html {
  background-color: #f0f6ff;
}

.modal-backdrop {
    /* bug fix - no overlay */    
    display: none;    
}

</style>
