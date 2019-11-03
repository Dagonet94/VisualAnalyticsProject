<template>
  <b-container>
    <b-row>
      <b-col class="12">
        <div id="yearlyHeatMap">
          <strong>Year, Location: </strong>
          <span class="reset" style="display: none;">Selected : <span class="filter"></span></span>
          <button class="reset" v-on:click="resetYearlyHeatMap" style="display: none;">reset</button>
          <div class="clearfix"></div>
        </div>
      </b-col>
    </b-row>
    <b-row>
      <b-col class="6">
        <div id="locChart">
          <strong>Location: </strong>
          <span class="reset" style="display: none;">Selected : <span class="filter"></span></span>
          <button class="reset" v-on:click="resetlocChart" style="display: none;">reset</button>
          <div class="clearfix"></div>
        </div>
      </b-col>
      <b-col class="6">
        <div id="measChart">
          <strong>Measure: </strong>
          <span class="reset" style="display: none;">Selected : <span class="filter"></span></span>
          <button class="reset" v-on:click="resetMeasChart" style="display: none;">reset</button>
          <div class="clearfix"></div>
        </div>
      </b-col>
    </b-row>
    <b-row>
      <b-col class="12">
        <div id="yearChart">
          <strong>Year: </strong>
          <span class="reset" style="display: none;">Selected : <span class="filter"></span></span>
          <button class="reset" v-on:click="resetYearChart" style="display: none;">reset</button>
          <div class="clearfix"></div>
        </div>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import crossfilter from 'crossfilter'
import * as d3 from 'd3'
import * as dc from 'dc'

let cf // crossfilter instance
// let all
// var numberFormat = d3.format('.2f')
let dYear // dimension for Year
let dLocation // dimension for RecordType
let dMeasure
let heatDim
let gLocation
let gMeasure
let gYear
let heatGroup

export default {
  name: 'InteractiveDashboard',
  props: {
    readyArray: Array
  },
  data () {
    return {
      data: this.readyArray,
      yearChart: [],
      measChart: [],
      yearlyHeatMap: [],
      locChart: [],
      year: {
        value: 'All'
      },
      location: {
        value: 'All'
      },
      measure: {
        value: 'All'
      },
      dataDim: [],
      locationDim: [],
      measureDim: []
    }
  },
  mounted () {
    cf = crossfilter(this.data)
    console.log(this.data)
    console.log('ciao')
    dLocation = cf.dimension(d => d.location)
    this.location.options = ['All'].concat(dLocation.group().reduceCount().all().map(v => v.key))
    console.log(this.location.options)
    dMeasure = cf.dimension(d => d.measure)
    dYear = cf.dimension(d => d.year)
    this.year.options = ['All'].concat(dYear.group().reduceCount().all().map(v => v.key))
    gLocation = dLocation.group()
    gMeasure = dMeasure.group()
    gYear = dYear.group()
    heatDim = cf.dimension(d => [d.year, d.location])
    heatGroup = heatDim.group()
    this.createGraphs()
  },
  methods: {
    resetYearChart () {
      this.yearChart.filterAll()
      dc.redrawAll()
    },
    resetlocChart () {
      this.locChart.filterAll()
      dc.redrawAll()
    },
    resetMeasChart () {
      this.measChart.filterAll()
      dc.redrawAll()
    },
    resetYearlyHeatMap () {
      this.yearlyHeatMap.filterAll()
      dc.redrawAll()
    },
    createGraphs () {
      this.yearChart = dc.barChart('#yearChart')
        .width(768)
        .height(380)
        .x(d3.scaleBand())
        .xUnits(dc.units.ordinal)
        .brushOn(false)
        .xAxisLabel('Year')
        .yAxisLabel('Count')
        .dimension(dYear)
        .barPadding(0.1)
        .outerPadding(0.05)
        .group(gYear)
      this.locChart = dc.rowChart('#locChart')
        .dimension(dLocation)
        .group(gLocation)
        .elasticX(true)
      this.measChart = dc.rowChart('#measChart')
        .dimension(dMeasure)
        .group(gMeasure)
        .data((group) => { return group.top(10) })
        .elasticX(true)
      this.yearlyHeatMap = dc.heatMap('#yearlyHeatMap')
        .dimension(heatDim)
        .group(heatGroup)
        .keyAccessor(function (d) { return d.key[0] })
        .valueAccessor(function (d) { return d.key[1] })
        .colorAccessor(function (d) { return +d.value })
        .title(function (d) {
          return 'Year:   ' + d.key[0] + '\n' +
            'Location:  ' + d.key[1] + '\n' +
            'Count: ' + (d.value)
        })
        .colors(['#ffffd9', '#edf8b1', '#c7e9b4', '#7fcdbb', '#41b6c4', '#1d91c0', '#225ea8', '#253494', '#081d58'])
        .calculateColorDomain()
      dc.renderAll()
    }
  }
}
</script>

<style>
  .dc-chart path.dc-symbol, .dc-legend g.dc-legend-item.fadeout {
    fill-opacity: 0.5;
    stroke-opacity: 0.5; }

  .dc-chart rect.bar {
    stroke: none;
    cursor: pointer; }
  .dc-chart rect.bar:hover {
    fill-opacity: .5; }

  .dc-chart rect.deselected {
    stroke: none;
    fill: #ccc; }

  .dc-chart .pie-slice {
    fill: #fff;
    font-size: 12px;
    cursor: pointer; }
  .dc-chart .pie-slice.external {
    fill: #000; }
  .dc-chart .pie-slice :hover, .dc-chart .pie-slice.highlight {
    fill-opacity: .8; }

  .dc-chart .pie-path {
    fill: none;
    stroke-width: 2px;
    stroke: #000;
    opacity: 0.4; }

  .dc-chart .selected path, .dc-chart .selected circle {
    stroke-width: 3;
    stroke: #ccc;
    fill-opacity: 1; }

  .dc-chart .deselected path, .dc-chart .deselected circle {
    stroke: none;
    fill-opacity: .5;
    fill: #ccc; }

  .dc-chart .axis path, .dc-chart .axis line {
    fill: none;
    stroke: #000;
    shape-rendering: crispEdges; }

  .dc-chart .axis text {
    font: 10px sans-serif; }

  .dc-chart .grid-line, .dc-chart .axis .grid-line, .dc-chart .grid-line line, .dc-chart .axis .grid-line line {
    fill: none;
    stroke: #ccc;
    shape-rendering: crispEdges; }

  .dc-chart .brush rect.selection {
    fill: #4682b4;
    fill-opacity: .125; }

  .dc-chart .brush .custom-brush-handle {
    fill: #eee;
    stroke: #666;
    cursor: ew-resize; }

  .dc-chart path.line {
    fill: none;
    stroke-width: 1.5px; }

  .dc-chart path.area {
    fill-opacity: .3;
    stroke: none; }

  .dc-chart path.highlight {
    stroke-width: 3;
    fill-opacity: 1;
    stroke-opacity: 1; }

  .dc-chart g.state {
    cursor: pointer; }
  .dc-chart g.state :hover {
    fill-opacity: .8; }
  .dc-chart g.state path {
    stroke: #fff; }

  .dc-chart g.deselected path {
    fill: #808080; }

  .dc-chart g.deselected text {
    display: none; }

  .dc-chart g.row rect {
    fill-opacity: 0.8;
    cursor: pointer; }
  .dc-chart g.row rect:hover {
    fill-opacity: 0.6; }

  .dc-chart g.row text {
    fill: black;
    font-size: 12px;
    cursor: pointer; }

  .dc-chart g.dc-tooltip path {
    fill: none;
    stroke: #808080;
    stroke-opacity: .8; }

  .dc-chart g.county path {
    stroke: #fff;
    fill: none; }

  .dc-chart g.debug rect {
    fill: #00f;
    fill-opacity: .2; }

  .dc-chart g.axis text {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    pointer-events: none; }

  .dc-chart .node {
    font-size: 0.7em;
    cursor: pointer; }
  .dc-chart .node :hover {
    fill-opacity: .8; }

  .dc-chart .bubble {
    stroke: none;
    fill-opacity: 0.6; }

  .dc-chart .highlight {
    fill-opacity: 1;
    stroke-opacity: 1; }

  .dc-chart .fadeout {
    fill-opacity: 0.2;
    stroke-opacity: 0.2; }

  .dc-chart .box text {
    font: 10px sans-serif;
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    pointer-events: none; }

  .dc-chart .box line {
    fill: #fff; }

  .dc-chart .box rect, .dc-chart .box line, .dc-chart .box circle {
    stroke: #000;
    stroke-width: 1.5px; }

  .dc-chart .box .center {
    stroke-dasharray: 3, 3; }

  .dc-chart .box .data {
    stroke: none;
    stroke-width: 0px; }

  .dc-chart .box .outlier {
    fill: none;
    stroke: #ccc; }

  .dc-chart .box .outlierBold {
    fill: red;
    stroke: none; }

  .dc-chart .box.deselected {
    opacity: 0.5; }
  .dc-chart .box.deselected .box {
    fill: #ccc; }

  .dc-chart .symbol {
    stroke: none; }

  .dc-chart .heatmap .box-group.deselected rect {
    stroke: none;
    fill-opacity: 0.5;
    fill: #ccc; }

  .dc-chart .heatmap g.axis text {
    pointer-events: all;
    cursor: pointer; }

  .dc-chart .empty-chart .pie-slice {
    cursor: default; }
  .dc-chart .empty-chart .pie-slice path {
    fill: #fee;
    cursor: default; }

  .dc-data-count {
    float: right;
    margin-top: 15px;
    margin-right: 15px; }
  .dc-data-count .filter-count, .dc-data-count .total-count {
    color: #3182bd;
    font-weight: bold; }

  .dc-legend {
    font-size: 11px; }
  .dc-legend .dc-legend-item {
    cursor: pointer; }

  .dc-hard .number-display {
    float: none; }

  div.dc-html-legend {
    overflow-y: auto;
    overflow-x: hidden;
    height: inherit;
    float: right;
    padding-right: 2px; }
  div.dc-html-legend .dc-legend-item-horizontal {
    display: inline-block;
    margin-left: 5px;
    margin-right: 5px;
    cursor: pointer; }
  div.dc-html-legend .dc-legend-item-horizontal.selected {
    background-color: #3182bd;
    color: white; }
  div.dc-html-legend .dc-legend-item-vertical {
    display: block;
    margin-top: 5px;
    padding-top: 1px;
    padding-bottom: 1px;
    cursor: pointer; }
  div.dc-html-legend .dc-legend-item-vertical.selected {
    background-color: #3182bd;
    color: white; }
  div.dc-html-legend .dc-legend-item-color {
    display: table-cell;
    width: 12px;
    height: 12px; }
  div.dc-html-legend .dc-legend-item-label {
    line-height: 12px;
    display: table-cell;
    vertical-align: middle;
    padding-left: 3px;
    padding-right: 3px;
    font-size: 0.75em; }

  .dc-html-legend-container {
    height: inherit; }

</style>
