<!-- Your HTML goes here -->
<template>
  <div>
    <h1>{{title}}</h1>
    <svg class="legends" height=40 width=450></svg>
    <svg :class="`scatterplot-svg-${chartId}`" :viewBox="viewBox">
      <g :transform="`translate(${margin.left}, ${margin.top})`">
        <g class="plot"></g>
        <g class="x-axis" :transform="`translate(0, ${height})`"></g>
        <g class="y-axis" :transform="`translate(${margin.left}, 0)`"></g>
      </g>

    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'ScatterPlot',
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
    attribX: {
      required: true,
      type: String,
    },
    labelX: {
      type: String
    },
    attribY: {
      required: true,
      type: String,
    },
    labelY: {
      type: String
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
      x: null,
      y: null,
      color: null,
      margin: {
        left: 75,
        top: 50,
        right: 20,
        bottom: 75,
      }
    };
  },
  mounted() {
      this.init()
  },
  methods: {
      init() {
          console.log(d3) // This can be savely removed
          // This will be called when the page loads.
          // You can load your data and setup D3 here
        this.x = d3.scaleLinear()
        this.y = d3.scaleSqrt()
        this.color = d3.scaleOrdinal()

        this.update()
      },
      update() {
        this.x.domain([d3.min(this.dataset, d => d.age ) , d3.max(this.dataset, d => d.age)])
            .range([this.margin.left, this.width + this.margin.right])
        this.y.domain(d3.extent(this.dataset, d => d.age1stCode))
            .range([this.height, 0]);
        this.color.domain(['Primary', 'Secondary',
                           'Associated', 'Bachelor',
                           'Master', 'Professional',
                           'Doctoral',
                           'Other'])
            .range(["#56ddff", "#2cb3fc",
                    "#0a80ff", "#1049de",
                    "#a15fff", "#7e59ff",
                    "#8200ff",
                    "#454343"]);

        this.renderScatter();
        this.renderXAxis();
        this.renderYAxis();
      },
    renderScatter(){
      var svg = d3.select(`.scatterplot-svg-${this.chartId}`)
          .select(".plot")

      const s = d3.scaleLinear()
          .domain([d3.min(this.dataset.map(d => d.yearsCode)), d3.max(this.dataset.map(d => d.yearsCode))])
          .range([1, 8]);

      svg.selectAll("circle")
          .data(this.dataset)
          .join("circle")
          .attr("transform", d => `translate(${this.x(d[this.attribX])},${this.y(d[this.attribY])})`)
          .attr("r", (d) => s(d.yearsCode ?? 0) )
          .attr("fill-opacity", "0.6")
          .attr('fill', (d) => this.color(d.edLevel))
    },
    renderXAxis() {
      let xAxis = d3.axisBottom(this.x).ticks(this.width / 80)

      d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.x-axis')
          .call(xAxis)
          .call(g => g.select(".domain").remove())
          .call(g => g.append("text")
              .attr("x", this.width - this.margin.right)
              .attr("y", -4)
              .attr("fill", "#000")
              .attr("font-weight", "bold")
              .attr("text-anchor", "end")
              .text(this.labelX))
    },
    renderYAxis() {
      let yAxis = d3.axisLeft(this.y)

      d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.y-axis')
          .call(yAxis)
          .call(g => g.select(".domain").remove())
          .call(g => g.select(".tick:last-of-type text").clone()
              .attr("x", 4)
              .attr("text-anchor", "start")
              .attr("font-weight", "bold")
              .text(this.labelY))
    },
  },
  computed: {
    viewBox() {
      return `0 0 ${this.width+this.margin.left+this.margin.right+ this.margin.right} ${this.height+this.margin.top+this.margin.bottom}`
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
/* Add your CSS here */
</style>