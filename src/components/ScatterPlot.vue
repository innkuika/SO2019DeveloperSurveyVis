<!-- Your HTML goes here -->
<template>
  <div>
    <h1>{{title}}</h1>
    <svg class="legends" height=45 width=600></svg>
    <svg :class="`scatterplot-svg-${chartId}`" :viewBox="viewBox">
      <g class="main-plot" :transform="`translate(${margin.left}, ${margin.top})`">
        <g class="plot"></g>
        <g class="x-axis" :transform="`translate(0, ${height})`"></g>
        <g class="y-axis" :transform="`translate(${margin.left}, 0)`"></g>
      </g>
    </svg>
    <h3>
      Dataset: Stack Overflow 2019 Developer Survey
    </h3>
    <p>
      [1] Bachelor or equivalent degree <br>
      [2] Master, Professional or Doctoral Degree <br>
      [3] No Degree or not applicable
    </p>
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
        this.x = d3.scaleLinear()
        this.y = d3.scaleSqrt()
        this.color = d3.scaleOrdinal()
        this.$emit('item-selected', null)
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
            .range(["#56ddff", "#56ddff",
                    "#0a80ff", "#0a80ff",
                    "#a15fff", "#a15fff",
                    "#a15fff",
                    "#ffc38b"]);

        this.renderScatter();
        this.renderXAxis();
        this.renderYAxis();
        this.renderLegend()
      },
    renderLegend() {
      var svg = d3.select(`.legends`)
      // Handmade legend
      svg.append("circle").attr("cx",6).attr("cy",10).attr("r", 6).style("fill", "#56ddff")
      svg.append("circle").attr("cx",230).attr("cy",10).attr("r", 6).style("fill", "#0a80ff")
      svg.append("circle").attr("cx",350).attr("cy",10).attr("r", 6).style("fill", "#a15fff")
      svg.append("circle").attr("cx",470).attr("cy",10).attr("r", 6).style("fill", "#ffc38b")
      svg.append("text").attr("x", 15).attr("y", 10).text("Elementary & Secondary").style("font-size", "15px").attr("alignment-baseline","middle")
      svg.append("text").attr("x", 239).attr("y", 10).text("Bachelor*").style("font-size", "15px").attr("alignment-baseline","middle")
      svg.append("text").attr("x", 359).attr("y", 10).text("Graduated*").style("font-size", "15px").attr("alignment-baseline","middle")
      svg.append("text").attr("x", 479).attr("y", 10).text("No Degree*").style("font-size", "15px").attr("alignment-baseline","middle")

      svg.append("circle").attr("cx",6).attr("cy",30).attr("r", 2).style("fill", "grey")
      svg.append("circle").attr("cx",200).attr("cy",30).attr("r", 12).style("fill", "grey")
      svg.append("text").attr("x", 15).attr("y", 30).text(`Coded for less than year`).style("font-size", "15px").attr("alignment-baseline","middle")
      svg.append("text").attr("x", 222).attr("y", 30).text(`Coded for ${d3.max(this.dataset.map(d => d.yearsCode))}  years`).style("font-size", "15px").attr("alignment-baseline","middle")
    },
    renderScatter(){
      var svg = d3.select(`.scatterplot-svg-${this.chartId}`)
          .select(".plot")

      const s = d3.scaleLinear()
          .domain([d3.min(this.dataset.map(d => d.yearsCode)), d3.max(this.dataset.map(d => d.yearsCode))])
          .range([2, 12]);

      svg.selectAll("circle")
          .data(this.dataset)
          .join("circle")
          .attr("transform", d => `translate(${this.x(d[this.attribX])},${this.y(d[this.attribY])})`)
          .attr("r", (d) => s(d.yearsCode ?? 0) )
          .attr("fill-opacity", "0.6")
          .attr('fill', (d) => this.color(d.edLevel))

      const brush = d3.brush()
          .on("start brush end", this.brushed)
          .on("end", this.brushedEnd)
      svg.call(brush);
    },
    brushedEnd({selection}) {
      let dot = d3.select(`.scatterplot-svg-${this.chartId}`)
          .select(".plot")
          .selectAll("circle")
      let value = [];
      if (selection) {
        const [[x0, y0], [x1, y1]] = selection;
        value = dot
            .attr("fill", "gray")
            //.filter(d => ((this.filterOption === "P" || "F" || "p" || "f" ) && (d.passOrFail === this.filterOption)) || (this.filterOption === "A"))
            .filter(d => x0 <= this.x(d[this.attribX]) && this.x(d[this.attribX]) < x1 && y0 <= this.y(d[this.attribY]) && this.y(d[this.attribY]) < y1)
            .attr('fill', (d) => this.color(d.edLevel))
            .data();
      } else {
        dot.attr('fill', (d) => this.color(d.edLevel));
      }
      d3.select(`.scatterplot-svg-${this.chartId}`)
          .property("value", value).dispatch("input");

      if (value.length === 0){
        this.$emit('item-selected', null)
      }
      else{
        this.$emit('item-selected', value)
      }


    },
    brushed({selection}){
      let dot = d3.select(`.scatterplot-svg-${this.chartId}`)
          .select(".plot")
          .selectAll("circle")
      let value = [];
      if (selection) {
        const [[x0, y0], [x1, y1]] = selection;
        value = dot
            .attr("fill", "gray") // Color everything grey
            // filter out the circles you want to color
            // .filter(d => ((this.filterOption === "P" || "F" || "p" || "f" ) && (d.passOrFail === this.filterOption)) || (this.filterOption === "A"))
            .filter(d => x0 <= this.x(d[this.attribX]) && this.x(d[this.attribX]) < x1 && y0 <= this.y(d[this.attribY]) && this.y(d[this.attribY]) < y1)
            // color them
            .attr('fill', (d) => this.color(d.edLevel)) //
            .data();
      } else {
        dot.attr('fill', (d) => this.color(d.edLevel));
      }
      d3.select(`.scatterplot-svg-${this.chartId}`)
          .property("value", value).dispatch("input");
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
.main-plot {
  background: white;
}
</style>