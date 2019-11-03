<template>
  <div id="test"></div>
</template>

<script>

import crossfilter from 'crossfilter'
import * as d3 from 'd3'
import * as dc from 'dc'

export default {
  name: 'ScatterPlot',
  props: {
    dataArray: Array
  },
  data () {
    return {
      interactions: this.dataArray
    }
  },
  mounted () {
    let data = this.interactions
    let chart = dc.seriesChart('#test')
    let subChart = function (c) {
      return dc.scatterPlot(c)
        .symbolSize(8)
        .highlightedSize(10)
    }
    let xValue = d => d.date
    data.forEach(function (x) {
      x.date = new Date(x.date)
    })
    let cf = crossfilter(data)
    let dim = cf.dimension(function (d) {
      return [d.date, d.eType]
    })
    let group = dim.group()
    chart
      .width(500)
      .height(500)
      .chart(subChart)
      .x(d3.scaleTime().domain(d3.extent(data, xValue))
        .range([0, innerWidth])
        .nice())
      .y(d3.scaleBand().domain([0, 4]))
      .shareTitle(false)
      .seriesAccessor(function (d) { return d.key[1] })
      .keyAccessor(function (d) { return +d.key[0] })
      .valueAccessor(function (d) { return +d.value })
      .clipPadding(10)
      .yAxisLabel('Type')
      .dimension(dim)
      .group(group)
      .elasticY(true)
      .mouseZoomable(true)
      .brushOn(true)
    dc.renderAll()
  }
}
</script>

<style>
</style>
