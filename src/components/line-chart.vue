<template>
  <button @click="fetchData">Clicke Me</button>
  <div class="chart">
    <svg width="960" height="500"></svg>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "LineChart",
  data() {
    return {
      svg: null,
      width: null,
      height: null,
      title: "Line Chart",
      xValue: null,
      yValue: null,
      xAxisLabel: "Time",
      yAxisLabel: "Temperature",
      margin: {
        top: 80,
        right: 40,
        bottom: 78,
        left: 105,
      },
    };
  },
  computed: {
    innerWidth() {
      return this.width - this.margin.left - this.margin.right;
    },
    innerHeight() {
      return this.height - this.margin.left - this.margin.right;
    },
  },
  methods: {
    fetchData() {
      d3.csv(
        "https://vizhub.com/curran/datasets/temperature-in-san-francisco.csv"
      ).then((data) => {
        data.forEach((d) => {
          d.temperature = +d.temperature;
          d.timestamp = new Date(d.timestamp);
          d.temperatureF = +d.temperature + 5;
        });
        console.log(data);

        this.render(data);
      });
    },

    render(data) {
      const xScale = d3
        .scaleTime()
        .domain(d3.extent(data, this.xValue))
        .range([0, this.innerWidth])
        .nice();

      const yScale = d3
        .scaleLinear()
        .domain(d3.extent(data, this.yValue))
        .range([this.innerHeight, 0])
        .nice();

      const g = this.svg
        .append("g")
        .attr("transform", `translate(${this.margin.left},${this.margin.top})`);

      const xAxis = d3
        .axisBottom(xScale)
        .tickSize(-this.innerHeight)
        .tickPadding(15);

      const yAxis = d3
        .axisLeft(yScale)
        .tickSize(-this.innerWidth)
        .tickPadding(10);

      const yAxisG = g.append("g").call(yAxis);

      yAxisG
        .append("text")
        .attr("class", "axis-label")
        .attr("y", -60)
        .attr("x", -this.innerHeight / 2)
        .attr("fill", "black")
        .attr("transform", `rotate(-90)`)
        .attr("text-anchor", "middle")
        .text(this.yAxisLabel);

      const xAxisG = g
        .append("g")
        .call(xAxis)
        .attr("transform", `translate(0,${this.innerHeight})`);

      xAxisG
        .append("text")
        .attr("class", "axis-label")
        .attr("y", 80)
        .attr("x", this.innerWidth / 2)
        .attr("fill", "black")
        .text(this.xAxisLabel);

      const lineGenerator = d3
        .line()
        .x((d) => xScale(this.xValue(d)))
        .y((d) => yScale(this.yValue(d)))
        .curve(d3.curveBasis);

      g.append("path")
        .attr("class", "line-path")
        .attr("d", lineGenerator(data));

      g.append("text")
        .attr("class", "title")
        .attr("y", -10)
        .text(this.title);
    },
  },
  mounted() {
    this.svg = d3.select("svg");
    this.width = +this.svg.attr("width");
    this.height = +this.svg.attr("height");
    this.xValue = (d) => d.timestamp;
    this.yValue = (d) => d.temperature;
  },
};
</script>

<style>
body {
  margin: 0px;
  overflow: hidden;
}

circle {
  fill: steelblue;
}

.line-path {
  fill: none;
  stroke: maroon;
  stroke-width: 2;
  stroke-linejoin: round;
}

text {
  font-family: sans-serif;
}

.tick text {
  font-size: 1.7em;
  fill: #635f5d;
}

.tick line {
  stroke: #c0c0bb;
}

.axis-label {
  font-size: 2.5em;
  fill: #8e8883;
}

.title {
  font-size: 2.2em;
  fill: #635f5d;
}
</style>
