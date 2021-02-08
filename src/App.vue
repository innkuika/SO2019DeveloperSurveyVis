<!-- This file will be used to layout and connect your views -->
<template>
  <div id="app">
      <!-- This is only a basic setup to display your views. Use HTML and CSS to create your layout using these basic commands -->
    <ScatterPlot
        class="scatter-plot"
        v-if="dataset"
        chartId="SO-survey1"
        title="Scatter Plot"
        :dataset="dataset"
        attribX="age"
        labelX="Age"
        attribY="age1stCode"
        labelY="Age First Code"
        :width="900"
        :height="800"/>
      <View2 />
      <View3 />
  </div>
</template>

<script>
// Try to use descriptive names like 'OverviewScatterPlot' or 'DetailLineChart'
import ScatterPlot from './components/ScatterPlot.vue'
import View2 from './components/View2.vue'
import View3 from './components/View3.vue'
import * as d3 from "d3";

export default {
  name: 'App',
  components: {
    ScatterPlot,
    View2,
    View3
  },
  data() {
    return {
      dataset: null
    }
  },
  created(){
    d3.csv("/data/survey_results_public.csv", d3.autoType).then((data) => {

      this.dataset = d3.map(data, (d) => {

        return {
          id: d["Respondent"],
          age1stCode: this.parseAge1stCode(d["Age1stCode"]),
          yearsCode: this.parseYearsCode(d["YearsCode"]),
          age: this.parseAge(d["Age"]),
          edLevel: this.parseEdLevel(d["EdLevel"])
        }
      }).filter(d => d.age > 0 && d.age1stCode > 0 && d.yearsCode > 0)
    });
  },
  methods: {
    parseAge(age){
      age = String(age)
      if (age.indexOf("NA") >= 0){
        return -1
      }
      else{
        return parseInt(age)
      }
    },
    parseYearsCode(yearsCode){
      yearsCode = String(yearsCode)
      if (yearsCode.indexOf("Less than") >= 0){
        return 0.5
      }
      else if (yearsCode.indexOf("More than") >= 0){
        return 51
      }
      else if (yearsCode.indexOf("NA") >= 0){
        return -1
      }
      else {
        return parseInt(yearsCode)
      }

    },
    parseAge1stCode(age1stCode){
      age1stCode = String(age1stCode)
      if (age1stCode.indexOf("Younger") >= 0){
        return 4
      }
      else if (age1stCode.indexOf("Older") >= 0){
        return 86
      }
      else if (age1stCode.indexOf("NA") >= 0){
        return -1
      }
      else {
        return parseInt(age1stCode)
      }
    },
    parseEdLevel(edLevel){
      const responseArr = ['Primary', 'Secondary', 'Associated', 'Bachelor', 'Master', 'Professional', 'doctoral']
      for(let i = 0; i < responseArr.length; i++){
        let index = edLevel.indexOf(responseArr[i])
        if (index >= 0) {
          return responseArr[i].charAt(0).toUpperCase() + responseArr[i].slice(1)
        }
      }
      return "Other"
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
