<!-- Your HTML goes here -->
<template>
  <div class="container">
    <div class="title">
      <h1 >{{ title }}</h1>
    </div>
    <button class="button1 button" v-on:click="state = 1">Do people today have a better life?</button>
    <button class="button2" v-on:click="state = 2">Are you the IT Person?</button>
    <div class="pie-chart-main">
      <svg :class="`pie-chart-svg-${chartId}`" :viewBox="viewBox">
        <g :transform="`translate(${margin.left}, ${margin.top})`">
          <g class="plot"></g>
        </g>
      </svg>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'PieChart', // Feel free to rename this and the file
  props: {
    // Feel free to add props to communicate between components
    chartId: {
      required: true,
      type: String
    },
    title: {
      type: String
    },
    dataset: {
      required: true,
      type: Array,
    },
    height: {
      required: true,
      type: Number,
    },
    width: {
      required: true,
      type: Number,
    },
  },
  data() {
    return {
      radius: null,
      state: 1,
      data1: null,
      data2: null,
      x: null,
      y: null,
      color: null,
      margin: {
        left: 10,
        top: -30,
        right: -10,
        bottom: 75,
      }
    };
  },
  mounted() {
    this.init()
  },
  watch: {
    dataset() {
      this.prepareData()
      this.update();
    },
    state() {
      this.update()
    }
  },
  methods: {
    init() {
      this.color = d3.scaleOrdinal()
      this.radius = Math.min(this.width, this.height) / 2 - 1.5 * this.margin.left
      this.prepareData()
      this.update()
    },
    update() {
      this.color.domain(['Yes', 'No', 'Sigh'])
          .range(["#84a1fa", "#f86b6b", "#9c6bf8"]);

      this.renderPieChart()
    },
    prepareData() {
      let betterLifeYesCount = 0
      let betterLifeNoCount = 0

      // itPerson counts
      let itPersonYesCount = 0
      let itPersonNoCount = 0
      let itPersonSighCount = 0

      for (let i = 0; i < this.dataset.length; i++) {
        if (this.dataset[i].betterLife === "Yes") {
          betterLifeYesCount++
        } else if (this.dataset[i].betterLife === "No") {
          betterLifeNoCount++
        }
        if (this.dataset[i].itPerson === "Yes") {
          itPersonYesCount++
        } else if (this.dataset[i].itPerson === "No") {
          itPersonNoCount++
        } else if (this.dataset[i].itPerson === "SIGH") {
          itPersonSighCount++
        }
      }

      this.data1 = [{name: "Yes", value: betterLifeYesCount}, {name: "No", value: betterLifeNoCount}]
      this.data2 = [{name: "Yes", value: itPersonYesCount}, {name: "No", value: itPersonNoCount}, {
        name: "Sigh",
        value: itPersonSighCount
      }]
    },
    renderPieChart() {
      let data = null
      if (this.state === 1) {
        data = this.data1
      } else {
        data = this.data2
      }

      const arc = d3.arc().innerRadius(this.radius * 0.6).outerRadius(this.radius);

      const pie = d3.pie()
          .padAngle(0.005)
          .sort(null)
          .value(d => d.value)

      const arcs = pie(data);

      var svg = d3.select(`.pie-chart-svg-${this.chartId}`)
          .select(".plot")

      svg.selectAll("text").remove()


      svg
          .selectAll("path")
          .data(arcs)
          .join("path")
          .attr("fill", d => this.color(d.data.name))
          .attr("d", arc)
          .append("title")
          .text(d => `${d.data.name}: ${d.data.value.toLocaleString()}`);

      svg.append("g")
          .attr("font-family", "sans-serif")
          .attr("font-size", 12)
          .attr("text-anchor", "middle")
          .selectAll("text")
          .data(arcs)
          .join("text")
          .attr("transform", d => `translate(${arc.centroid(d)})`)
          .call(text => text.append("tspan")
              .attr("y", "-0.4em")
              .attr("font-weight", "bold")
              .text(d => d.data.name))
          .call(text => text.filter(d => (d.endAngle - d.startAngle) > 0.25).append("tspan")
              .attr("x", 0)
              .attr("y", "0.7em")
              .attr("fill-opacity", 0.7)
              .text(d => d.data.value.toLocaleString()));

      return svg.node();

    }
  },
  computed: {
    viewBox() {
      return `${(-this.width + this.margin.left + this.margin.right + this.margin.right)/2} ${-(this.height + this.margin.top + this.margin.bottom)/2} ${this.width + this.margin.left + this.margin.right + this.margin.right} ${this.height + this.margin.top + this.margin.bottom}`
    },

  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* Add your CSS here */
.container {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
  display: grid;
  grid-template-rows: 10vh 10vh 10vh;
  grid-template-columns: 1fr 7fr 1fr 1fr;
  grid-template-areas:
    ". title title ."
    ". main bt1 bt1"
    ". main bt2 bt2";
}

.pie-chart-main {
  grid-area: main;
  justify-self: stretch;
}

.button1 {
  grid-area: bt1;
  background: #ff926e;
  border: #ff7b43;
  border-radius: 10px;
  height: 80px;
}

.button2 {
  grid-area: bt2;
  background: #ffcc9f;
  border: lightsalmon;
  border-radius: 10px;
  height: 80px;
}

.title {
  grid-area: title;
}

</style>