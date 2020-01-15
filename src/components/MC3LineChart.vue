<template>
  <b-container>
    <b-col class="12">
    <div class="graphTitle">Company overview</div>
    <div class="select">
    <select v-on:change="change" id="selectButton"></select>
      </div>
    <div class="graph" id="lineChart"></div>
    </b-col>
  </b-container>
</template>

<script>
import * as d3 from 'd3'
let margin = {top: 10, right: 30, bottom: 30, left: 60}
let width = 580 - margin.left - margin.right
let height = 520 - margin.top - margin.bottom
let selectedGroup = null
let x = null
let y = null
let xAxis = null
let yAxis = null
let myColor = null
let svg = null

export default {
  name: 'MC3StackedArea',
  data () {
    return {
      lineChart: null,
      array: null
    }
  },
  mounted () {
    // let parseTime = d3.timeParse('%Y-%m-%d')
    fetch('/static/data/interactionsByDay.json')
      .then(res => res.json())
      .then((res) => {
        const measure = res.map((d) => {
          const r = {
            type: d.Etype,
            value: +d.value,
            sampleDate: new Date(d.date)
          }
          return r
        })
        this.array = measure.sort(function (a, b) {
          return new Date(a.sampleDate) - new Date(b.sampleDate)
        })
        console.log(measure)
        let allGroup = d3.map(measure, function (d) {
          return (d.type)
        }).keys()
        d3.select('#selectButton')
          .selectAll('myOptions')
          .data(allGroup)
          .enter()
          .append('option')
          .text(function (d) {
            return d
          }) // text showed in the menu
          .attr('value', function (d) {
            return d
          })
        svg = d3.select('#lineChart')
          .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
          .append('g')
          .attr('class', 'effect8')
          .attr('transform',
            'translate(' + margin.left + ',' + margin.top + ')')
        x = d3.scaleTime()
          .domain(d3.extent(measure, function (d) {
            return d.sampleDate
          }))
          .range([0, width])
        xAxis = d3.axisBottom(x)
        svg.append('g')
          .attr('transform', 'translate(0,' + height + ')')
          .call(xAxis)

        y = d3.scaleLinear()
          .domain([0, d3.max(measure, function (d) {
            return d.value
          })])
          .range([height, 0])
        yAxis = d3.axisLeft(y)
        svg.append('g')
          .call(yAxis)
          .attr('id', 'ciao')
        myColor = d3.scaleOrdinal()
          .domain(allGroup)
          .range(d3.schemeSet2)
        console.log(svg)
        // Draw the line
        this.line = svg
          .append('g')
          .append('path')
          .datum(measure.filter(function (d) {
            return d.type === allGroup[0]
          }).sort(function (a, b) {
            return new Date(a.sampleDate) - new Date(b.sampleDate)
          }))
          .attr('d', d3.line()
            .x(function (d) {
              return x(d.sampleDate)
            })
            .y(function (d) {
              return y(d.value)
            })
          )
          .attr('stroke', function (d) {
            return myColor('call')
          })
          .style('stroke-width', 1.5)
          .style('fill', 'none')
      })
  },
  updated () {
  },
  methods: {
    change () {
      selectedGroup = d3.select('#selectButton').property('value')
      console.log(selectedGroup)

      let dataFilter = this.array.filter(function (d) {
        return d.type === selectedGroup
      })
      y.domain(d3.extent(dataFilter, function (d) {
        return d.value
      }))
        .range([height, 0])
      svg.select('#ciao')
        .transition()
        .duration(1000)
        .call(d3.axisLeft(y))
      this.line
        .datum(dataFilter)
        .transition()
        .duration(1000)
        .attr('d', d3.line()
          .x(function (d) { return x(d.sampleDate) })
          .y(function (d) { return y(d.value) })
        )
        .attr('stroke', function (d) { return myColor(selectedGroup) })
    }
  }
}
// Give these new data to update line
/* this.line
      .datum(dataFilter)
      .transition()
      .duration(1000)
      .attr('d', d3.line()
        .x(function (d) { return this.x(d.sampledate) })
        .y(function (d) { return this.y(+d.value) })
      )
      .attr('stroke', function (d) { return this.myColor(selectedGroup) }) */
</script>

<style scoped>
  .effect8
  {
    position:relative;
    -webkit-box-shadow:0 1px 4px rgba(0, 0, 0, 0.3), 0 0 40px rgba(0, 0, 0, 0.1) inset;
    -moz-box-shadow:0 1px 4px rgba(0, 0, 0, 0.3), 0 0 40px rgba(0, 0, 0, 0.1) inset;
    box-shadow:0 1px 4px rgba(0, 0, 0, 0.3), 0 0 40px rgba(0, 0, 0, 0.1) inset;
  }
  .effect8:before, .effect8:after
  {
    content:"";
    position:absolute;
    z-index:-1;
    -webkit-box-shadow:0 0 20px rgba(0,0,0,0.8);
    -moz-box-shadow:0 0 20px rgba(0,0,0,0.8);
    box-shadow:0 0 20px rgba(0,0,0,0.8);
    top:10px;
    bottom:10px;
    left:0;
    right:0;
    -moz-border-radius:100px / 10px;
    border-radius:100px / 10px;
  }
  .effect8:after {
    right: 10px;
    left: auto;
    -webkit-transform: skew(8deg) rotate(3deg);
    -moz-transform: skew(8deg) rotate(3deg);
    -ms-transform: skew(8deg) rotate(3deg);
    -o-transform: skew(8deg) rotate(3deg);
    transform: skew(8deg) rotate(3deg);
  }
  .graph {
    box-shadow: 0 19px 38px rgba(0,0,0,0.30), 0 15px 12px rgba(0,0,0,0.22);
    margin-bottom: 30px;
    border-bottom-left-radius: 10px;
    border-bottom-right-radius: 10px;
    size: auto}
  .graphTitle {
    color: white;
    margin-top: 30px;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    background-color: #17a2b8;
  }
  .select {
    background-color: #17a2b8;

  }

</style>
