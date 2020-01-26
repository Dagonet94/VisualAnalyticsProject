<template>
  <b-container>
    <b-row>
      <b-col class="12">
    <div id="sliderContainer">
      <vue-slider
        v-model="sliderValue"
        :min="min"
        :max="max"
        :step="step"
        :width=1000
        :tooltip-formatter="formatter"
        :tooltip= "'always'"
        :lazy = "true"
        :enable-cross= "false"
      ></vue-slider>
    </div>
      </b-col>
    </b-row>
    <b-row>
    <b-col class="12">
      <div class="graphTitle">Interactive LineChart</div>
      <div class="select">
        <select v-on:change="change" id="selectButton"></select>
        <select v-on:change="change" id="selectButtonLoc"></select>
      </div>
      <div class="graph" id="lineChart"></div>
    </b-col>
    </b-row>
  </b-container>
</template>

<script>
import * as d3 from 'd3'
import * as smooth from 'array-smooth'
let margin = {top: 10, right: 30, bottom: 30, left: 60}
let width = 580 - margin.left - margin.right
let height = 520 - margin.top - margin.bottom
let selectedGroup = null
let selectedLoc = null
let x = null
let y = null
let xAxis = null
let yAxis = null
let myColor = null
let svg = null
let parseTime = d3.timeParse('%Y-%m-%d')
let formatTime = d3.timeFormat('%d/%m/%Y')

export default {
  name: 'MC2LineChartSniper',
  props: {
    readyArray: Array
  },
  data () {
    return {
      sliderValue: [parseTime('1998-01-11').getTime(), parseTime('2016-12-31').getTime()],
      formatter: v => `${formatTime((new Date(v)))}`,
      dataArray: this.readyArray,
      max: parseTime('2016-12-31').getTime(),
      min: parseTime('1998-01-11').getTime(),
      step: 86400000,
      lineChart: null,
      array: null,
      temp: null
    }
  },
  mounted () {
    // let parseTime = d3.timeParse('%Y-%m-%d')
    this.array = this.dataArray.sort(function (a, b) {
      return new Date(a.sampleDate) - new Date(b.sampleDate)
    })
    let prova = []
    this.array.forEach(function (d) {
      prova.push(d.value)
    })
    this.temp = smooth(prova, 2)
    let locGroup = d3.map(this.array, function (d) {
      return (d.location)
    }).keys()
    let allGroup = d3.map(this.array, function (d) {
      return (d.measure)
    }).keys()
    d3.select('#selectButtonLoc')
      .selectAll('myOptions')
      .data(locGroup)
      .enter()
      .append('option')
      .text(function (d) {
        return d
      }) // text showed in the menu
      .attr('value', function (d) {
        return d
      })
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
      .domain(d3.extent(this.dataArray, function (d) {
        return d.sampleDate
      }))
      .range([0, width])
    xAxis = d3.axisBottom(x)
    svg.append('g')
      .attr('transform', 'translate(0,' + height + ')')
      .call(xAxis)
      .attr('id', 'dateAx')
    y = d3.scaleLinear()
      .domain([0, d3.max(this.dataArray.filter(function (d) {
        return (d.measure === allGroup[0] && d.location === locGroup[0])
      }), function (d) {
        return d.value
      })])
      .range([height, 0])
    yAxis = d3.axisLeft(y)
    svg.append('g')
      .call(yAxis)
      .attr('id', 'ciao')
    d3.select('#ciao')
      .append('text')
      .attr('id', 'ylabel')
      .attr('class', 'y axisLabel')
      .attr('transform', 'rotate(-90)')
      .attr('y', -60)
      .attr('x', -170)
      .attr('font-size', '20px')
      .attr('text-anchor', 'middle')
      .attr('fill', 'black')
      .text(d3.select('#selectButton').property('value'))
    myColor = d3.scaleOrdinal()
      .domain(allGroup)
      .range(d3.schemeSet2)
    // Draw the line
    this.line = svg
      .append('g')
      .append('path')
      .datum(this.dataArray.filter(function (d) {
        return (d.measure === allGroup[0] && d.location === locGroup[0])
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
        return myColor('Water Temperature')
      })
      .style('stroke-width', 1.5)
      .style('fill', 'none')
  },
  updated () {
  },
  methods: {
    change () {
      let startDate = this.sliderValue[0]
      let stopDate = this.sliderValue[1]
      selectedGroup = d3.select('#selectButton').property('value')
      selectedLoc = d3.select('#selectButtonLoc').property('value')
      d3.select('#ylabel').remove()
      d3.select('#ciao')
        .append('text')
        .attr('class', 'y axisLabel')
        .attr('id', 'ylabel')
        .attr('transform', 'rotate(-90)')
        .attr('y', -60)
        .attr('x', -170)
        .attr('font-size', '20px')
        .attr('text-anchor', 'middle')
        .attr('fill', 'black')
        .text(selectedGroup)
      let dataFilter = this.dataArray.filter(function (d) {
        return (d.measure === selectedGroup && d.location === selectedLoc)
      })
      dataFilter = dataFilter.filter(function (d) {
        return (d.sampleDate.getTime() > startDate && d.sampleDate.getTime() < stopDate)
      })
      y.domain(d3.extent(dataFilter, function (d) {
        return d.value
      }))
        .range([height, 0])
      x.domain(d3.extent(dataFilter, function (d) {
        return d.sampleDate
      }))
        .range([0, width])
      svg.select('#dateAx')
        .transition()
        .duration(1000)
        .call(d3.axisBottom(x))
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
  },
  watch: {
    sliderValue () {
      this.change()
      // this.createNetwork(returnList[0], returnList[1])
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
