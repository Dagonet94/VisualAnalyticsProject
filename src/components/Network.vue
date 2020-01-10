<template>
  <b-container>
    <b-row class="justify-content-md-center">
      <b-col>
        <label> Solo azioni sospette</label>
        <input type="checkbox" v-model="suspectActionCheck">
        <b-form-group v-if="visible===true" label="Select a suspect">
          <b-form-select
            v-model="value"
            :options="options"
          ></b-form-select>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <svg width="1000" height="1000"></svg>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import * as d3 from 'd3'
let returnList
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
      suspectActionCheck: true,
      visible: true,
      simulation: null,
      nodes: this.nodesArray,
      links: this.linksArray,
      suspectDegreeSource: 1,
      suspectDegreeDest: 1,
      suspectDegreeSourceExtend: 1,
      suspectAction: 'Yes',
      value: '',
      options: []
    }
  },
  mounted () {
    this.createOption()
    // this.createNetwork(this.nodes, this.dSource.top(1000))
    returnList = this.updateData()
    this.createNetwork(returnList[0], returnList[1])
  },

  methods: {
    createNetwork (node, link) {
      d3.select('svg').selectAll('*').remove()
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
          return color(d.suspectDegree)
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

    updateData: function () {
      let linkFiltered = []
      let test = this.value
      let listTemp = []
      let suspectDegreeDest = this.suspectDegreeDest
      let suspectActionCheck = this.suspectActionCheck
      let suspectAction
      if (suspectActionCheck === true) {
        suspectAction = 'Yes'
      }
      if (suspectActionCheck === false) {
        suspectAction = 'No'
      }
      listTemp.push(test)
      let linksArray = this.linksArray
      console.log(linksArray)
      console.log(test)
      // Solo i contatti del sospetto
      linksArray.forEach(function (element) {
        if (test === element.source & element.suspectDegreeDest <= suspectDegreeDest & element.suspectAction === suspectAction) {
          linkFiltered.push(element)
          listTemp.push(element.target)
          console.log('sorgente')
        }
        if (test === element.target & element.suspectDegreeSource <= suspectDegreeDest & element.suspectAction === suspectAction) {
          linkFiltered.push(element)
          listTemp.push(element.source)
          console.log('destinatario')
        }
      })
      listTemp = [...new Set(listTemp)]
      // Contatti dei contatti
      let suspectDegreeStack = this.suspectDegreeSourceExtend
      linksArray.forEach(function (element) {
        listTemp.forEach(function (element2) {
          if (element2 === element.source & element.suspectDegreeDest <= suspectDegreeStack & element.suspectAction === suspectAction) {
            linkFiltered.push(element)
          }
        })
      })
      let nodesArray = this.nodesArray
      let nodeFiltered = []
      nodesArray.forEach(function (element) {
        linkFiltered.forEach(function (element2) {
          if (element2.source === element.id | element2.target === element.id) {
            nodeFiltered.push(element)
          }
        })
      })
      nodeFiltered = [...new Set(nodeFiltered)]
      console.log(linkFiltered)
      console.log(nodeFiltered)
      returnList = [nodeFiltered, linkFiltered]
      return returnList
    },

    createOption: function () {
      let optionArray = []
      let suspectDegree = this.suspectDegreeSource
      this.nodesArray.forEach(function (element) {
        if (element.suspectDegree <= suspectDegree) {
          optionArray.push(element.id)
        }
      })

      this.options = [...new Set(optionArray)]
    }
  },
  watch: {
    suspectActionCheck () {
      returnList = this.updateData()
      this.createNetwork(returnList[0], returnList[1])
    },

    value () {
      returnList = this.updateData()
      this.createNetwork(returnList[0], returnList[1])
    }
  }
}
</script>

<style scoped>
</style>
