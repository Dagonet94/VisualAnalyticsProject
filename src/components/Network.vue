<template>
  <b-container>
    <b-row class="justify-content-md-center">
      <b-col>
        <b-form-group label="Select a suspect">
          <b-form-select
            v-model="source.value"
            :options="source.options"
            name="buttonsLocations"
            buttons
          ></b-form-select>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <svg width="500" height="500"></svg>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import * as d3 from 'd3'
import crossfilter from 'crossfilter'

let width = 500
let height = 500
let color = d3.scaleOrdinal(d3.schemePaired)

export default {
  name: 'Network',
  props: {
    nodesArray: Array,
    linksArray: Array
  },
  data () {
    return {
      simulation: null,
      nodes: this.nodesArray,
      links: this.linksArray,
      source: {
        value: '',
        options: []
      },
      dSource: '',
      filtered: []
    }
  },
  mounted () {
    let cf = crossfilter(this.links)
    this.dSource = cf.dimension(d => d.source)
    this.source.options = this.dSource.group().reduceCount().all().map(v => v.key)
    this.source.value = this.source.options[0]
    // this.createNetwork(this.nodes, this.dSource.top(1000))
  },
  updated () {
    this.refreshChart()
    this.updateChart()
  },
  methods: {
    updateChart () {
      d3.select('svg').selectAll('*').remove()
      let nodi = this.nodes
      console.log(this.dSource.top(3))
      let linkFiltered = this.dSource.filter(d => d === this.source.value).top(1000)
      let sourceFiltered = linkFiltered.map((d) => {
        if (d.target.id) {
          return d.target.id
        } else {
          return d.target
        }
      })
      sourceFiltered.push(this.source.value)
      let nodeFiltered = nodi.filter(f => sourceFiltered.includes(f.id))
      this.createNetwork(nodeFiltered, linkFiltered)
    },
    createNetwork (node, link) {
      this.simulation = d3.forceSimulation()
        .force('center', d3.forceCenter(width / 2, height / 2))
        .force('charge', d3.forceManyBody().strength(-50))
        .force('collide', d3.forceCollide(10).strength(0.9))
        .force('link', d3.forceLink().id(function (node) {
          return node.id
        }))
      let linked = d3.select('svg').append('g')
        .attr('class', 'links')
        .selectAll('line')
        .data(link)
        .enter().append('line')
        .attr('stroke-width', function (d) {
          return d.value
        })
        .attr('stroke', 'grey')

      let noded = d3.select('svg').append('g')
        .attr('class', 'nodes')
        .selectAll('circle')
        .data(node)
        .enter().append('circle')
        .attr('r', 5)
        .attr('fill', function (d) {
          return color(d.group)
        })
        .call(d3.drag()
          .on('start', this.dragstarted)
          .on('drag', this.dragged)
          .on('end', this.dragended)
        )
      noded.append('title')
        .text(function (d) {
          return d.id
        })
      this.simulation
        .nodes(node)
        .on('tick', ticked)
      this.simulation.force('link')
        .links(link)

      // Dynamically update the position of the nodes/links as time passes
      function ticked () {
        linked
          .attr('x1', function (d) {
            return d.source.x
          })
          .attr('y1', function (d) {
            return d.source.y
          })
          .attr('x2', function (d) {
            return d.target.x
          })
          .attr('y2', function (d) {
            return d.target.y
          })

        noded
          .attr('cx', function (d) {
            return d.x
          })
          .attr('cy', function (d) {
            return d.y
          })
      }
    },
    dragstarted (d) {
      if (!d3.event.active) this.simulation.alphaTarget(0.7).restart()
      d.fx = d.x
      d.fy = d.y
    },
    dragged (d) {
      d.fx = d3.event.x
      d.fy = d3.event.y
    },
    dragended (d) {
      if (!d3.event.active) this.simulation.alphaTarget(0)
      d.fx = null
      d.fy = null
    },
    refreshChart () {
      this.dSource = this.dSource.filterAll()
      this.filtered = this.dSource.filter(d => d === this.source.value)
    }
  }
}
</script>

<style scoped>

</style>
