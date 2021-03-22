<!-- This file will be used to layout and connect your views -->
<template>
  <div id="app">
    <ScatterPlot
        class="scatter-plot"
        v-if="dataset"
        chartId="SO-survey1"
        title="Did People All Quit School?"
        :dataset="dataset"
        attribX="age"
        labelX="Age"
        attribY="age1stCode"
        labelY="Age First Code"
        :width="900"
        :height="800"
        @item-selected="circleSelected"
    />
    <WorldMap
        class="world-map"
        v-if="selectedData"
        chartId="SO-survey2"
        title="How Much Time Do People Spend On Working?"
        :dataset="selectedData"
        :width="900"
        :height="400"
        :world="world"
    />
    <PieChart
        class="pie-chart"
        v-if="selectedData"
        chartId="SO-survey3"
        title="What's In People's Mind?"
        :dataset="selectedData"
        :width="400"
        :height="250"
    />
  </div>
</template>

<script>
// Try to use descriptive names like 'OverviewScatterPlot' or 'DetailLineChart'
import ScatterPlot from './components/ScatterPlot.vue'
import WorldMap from "@/components/WorldMap";
import PieChart from './components/PieChart.vue'
import * as d3 from "d3";


export default {
  name: 'App',
  components: {
    WorldMap,
    ScatterPlot,
    PieChart
  },
  data() {
    return {
      dataset: null,
      world: null,
      selectedData: null
    }
  },
  created() {
    d3.json("/data/countries-50m.json", d3.autoType).then((data) => {
      this.world = data
    })
    d3.csv("/data/survey_results_public.csv", d3.autoType).then((data) => {
      this.dataset = d3.map(data, (d) => {
        return {
          id: d["Respondent"],
          age1stCode: this.parseAge1stCode(d["Age1stCode"]),
          yearsCode: this.parseYearsCode(d["YearsCode"]),
          age: this.parseAge(d["Age"]),
          edLevel: this.parseEdLevel(d["EdLevel"]),
          country: this.parseCountry(d["Country"]),
          workHour: this.parseWorkHour(d["WorkWeekHrs"]),
          betterLife: d["BetterLife"],
          itPerson: this.parseITPerson(d["ITperson"])
        }
      }).filter((d, i) => i % 100 === 0 && d.age > 0 && d.age1stCode > 0 && d.yearsCode > 0 && d.age > d.age1stCode && d.age < 80)
    });
  },
  methods: {
    circleSelected(selection) {
      if (selection === null) {
        selection = this.dataset
      }
      this.selectedData = selection
    },
    parseITPerson(itPerson) {
      if (itPerson === "Also Yes") {
        return "Yes"
      } else if (itPerson === "Fortunately, someone else has that title") {
        return "No"
      } else {
        return itPerson
      }
    },
    parseAge(age) {
      age = String(age)
      if (age.indexOf("NA") >= 0) {
        return -1
      } else {
        return parseInt(age)
      }
    },
    parseYearsCode(yearsCode) {
      yearsCode = String(yearsCode)
      if (yearsCode.indexOf("Less than") >= 0) {
        return 0.5
      } else if (yearsCode.indexOf("More than") >= 0) {
        return 51
      } else if (yearsCode.indexOf("NA") >= 0) {
        return -1
      } else {
        return parseInt(yearsCode)
      }

    },
    parseAge1stCode(age1stCode) {
      age1stCode = String(age1stCode)
      if (age1stCode.indexOf("Younger") >= 0) {
        return 4
      } else if (age1stCode.indexOf("Older") >= 0) {
        return 86
      } else if (age1stCode.indexOf("NA") >= 0) {
        return -1
      } else {
        return parseInt(age1stCode)
      }
    },
    parseEdLevel(edLevel) {
      const responseArr = ['Primary', 'Secondary', 'Associated', 'Bachelor', 'Master', 'Professional', 'doctoral']
      for (let i = 0; i < responseArr.length; i++) {
        let index = edLevel.indexOf(responseArr[i])
        if (index >= 0) {
          return responseArr[i].charAt(0).toUpperCase() + responseArr[i].slice(1)
        }
      }
      return "Other"
    },
    parseCountry(country) {
      if (country === "United States") {
        return "United States of America"
      } else if (country === "Russian Federation") {
        return "Russia"
      } else if (country === "Antigua and Barbuda") {
        return "Antigua and Barb."
      }
      return country
    },
    parseWorkHour(workHour) {
      if (workHour === 'NA') {
        return -1
      } else {
        return parseInt(workHour)
      }
    }
  }

}
</script>

<style>
body {background-color: whitesmoke;}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: grid;
  grid-template-rows: 50vh 40vh;
  grid-template-columns: 1fr 10fr 10fr 1fr;
  grid-template-areas:
    ". overview detail1 ."
    ". overview detail2 .";
}

.scatter-plot {
  grid-area: overview;
}

.world-map {
  grid-area: detail1;
}
.pie-chart {
  grid-area: detail2;
}
</style>
