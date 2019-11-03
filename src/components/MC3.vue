<template>
  <div v-if="childDataLoaded">
  <Network :linksArray="linksArray" :nodesArray="nodesArray"></Network>
  </div>
</template>

<script>
import ScatterPlot from '@/components/ScatterPlot'
import Network from '@/components/Network'
// import * as d3 from 'd3'
export default {
  name: 'MC3',
  components: {
    ScatterPlot,
    Network
  },
  data () {
    return {
      childDataLoaded: false,
      eType: '',
      source: '',
      target: '',
      date: '',
      linksArray: [],
      nodesArray: [],
      nodes: {
        id: null,
        group: null
      },
      links: {
        source: null,
        target: null
      }
    }
  },
  mounted () {
    fetch('/static/data/onlySusp.json')
      .then(res => res.json())
      .then((res) => {
        const node = res.nodes.map((d) => {
          const n = {
            id: d.id,
            group: d.group
          }
          return n
        })
        const link = res.links.map((d) => {
          const n = {
            source: d.source,
            target: d.target
          }
          return n
        })
        this.linksArray = link
        this.nodesArray = node
        this.childDataLoaded = true
      })
  }
}
</script>

<style scoped>

</style>
