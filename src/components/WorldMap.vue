<!-- Your HTML goes here -->
<template>
  <div>
    <h1>{{ title }}</h1>
    <svg class="legends" height=45 width=600></svg>
    <svg :class="`world-map-svg-${chartId}`" :viewBox="viewBox">
      <g :transform="`translate(${margin.left}, ${margin.top})`">
        <g class="plot"></g>
      </g>

    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import * as topojson from 'topojson-client'

export default {
  name: 'WorldMap', // Feel free to rename this and the file
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
    world: {
      required: true,
      type: Object
    }
  },
  data() {
    return {
      country: null,
      data: null,
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
  watch: {
    dataset() {
      this.update()
    }
  },
  methods: {
    init() {
      this.country = topojson.feature(this.world, this.world.objects.countries)
      this.color = d3.scaleSequential()
      this.update()
    },
    update() {
      this.parseDataset()
      this.color
          //.domain(d3.extent(Object.values(this.data)))
          .domain([10, 80])
          .interpolator(d3.interpolateYlGnBu)
          .unknown("#ccc")

      this.renderMap()
    },
    parseDataset() {
      // filter data passed in based on work hour
      this.data = this.dataset.filter(d => d.workHour > 10 && d.workHour < 110)

      let data = {}
      for (let i = 0; i < this.data.length; i++) {
        let key = this.data[i].country
        let value = this.data[i].workHour
        if (data[key] === undefined) {
          data[key] = [value]
        } else {
          data[key].push(value)
        }
      }

      let final_data = {}
      for (var key in data) {
        let value = data[key]
        const average = arr => arr.reduce((p, c) => p + c, 0) / arr.length;

        final_data[key] = average(value)
      }
      this.data = final_data

    },
    renderMap() {
      const projection = d3.geoEqualEarth()
      const outline = ({type: "Sphere"})
      const path = d3.geoPath(projection)

      const svg = d3.select(`.world-map-svg-${this.chartId}`)
          .select(".plot")

      const tooltipDiv = d3.select("body").append("div")
          .attr("class", "tooltip")
          .style("opacity", 0);


      const defs = svg.append("defs");

      defs.append("path")
          .attr("id", "outline")
          .attr("d", path(outline));

      defs.append("clipPath")
          .attr("id", "clip")
          .append("use")
          .attr("xlink:href", new URL("#outline", location));

      const g = svg.append("g")
          .attr("clip-path", `url(${new URL("#clip", location)})`);

      g.append("use")
          .attr("xlink:href", new URL("#outline", location))
          .attr("fill", "white");


      g.append("g")
          .selectAll("path")
          .data(this.country.features)
          .join("path")
          .attr("fill", d => {
            return this.color(this.data[d.properties.name])
          })
          .attr("d", path)
          .on("mouseover", (event, d) => {
            tooltipDiv.transition()
                .duration(200)
                .style("opacity", .9);
            tooltipDiv.html(d.properties.name + " " + Math.round(this.data[d.properties.name]) + "h")
                .style("left", (event.pageX) + "px")
                .style("top", (event.pageY - 28) + "px");
          })
          .on("mouseout", function () {
            tooltipDiv.transition()
                .duration(500)
                .style("opacity", 0);
          })``


      g.append("path")
          .datum(topojson.mesh(this.world, this.world.objects.countries, (a, b) => a !== b))
          .attr("fill", "none")
          .attr("stroke", "white")
          .attr("stroke-linejoin", "round")
          .attr("d", path);


      return svg.node();
    }
  },
  computed: {
    viewBox() {
      return `0 0 ${this.width + this.margin.left + this.margin.right + this.margin.right} ${this.height + this.margin.top + this.margin.bottom}`
    },

  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
/* Add your CSS here */
.tooltip {
  position: absolute;
  text-align: center;
  width: 80px;
  height: 40px;
  padding: 2px;
  font: 12px sans-serif;
  background: seashell;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
}
</style>