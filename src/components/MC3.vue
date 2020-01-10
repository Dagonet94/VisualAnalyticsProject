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
      linksArray: [],
      nodesArray: [],
      nodes: {
        id: null,
        first: null,
        last: null,
        value: null,
        nameSurname: null,
        suspectDegree: null

      },
      links: {
        source: null,
        target: null,
        Etype: null,
        Time: null,
        nameSurnameDest: null,
        nameSurnameSource: null,
        suspectDegreeDest: null,
        suspectDegreeSource: null,
        suspectAction: null
      }
    }
  },
  mounted () {
    fetch('/static/data/dataFinal.json')
      .then(res => res.json())
      .then((res) => {
        const node = res.Nodes.map((d) => {
          const n = {
            id: +d.ID,
            first: d.first,
            last: d.last,
            suspectDegree: +d.suspectDegree,
            nameSurname: d.nameSurname

          }
          return n
        })
        const link = res.Link.map((d) => {
          const n = {
            source: +d.Source,
            target: +d.Dest,
            Etype: +d.Etype,
            Time: +d.Time,
            nameSurnameDest: d.nameSurnameDest,
            nameSurnameSource: d.nameSurnameSource,
            suspectDegreeDest: +d.suspectDegreeDest,
            suspectDegreeSource: +d.suspectDegreeSource,
            suspectAction: d.suspectAction

          }
          return n
        })
        this.linksArray = link
        this.nodesArray = node
        console.log(node)
        console.log(link)
        this.childDataLoaded = true
      })
  }
}
</script>

<style scoped>

</style>
