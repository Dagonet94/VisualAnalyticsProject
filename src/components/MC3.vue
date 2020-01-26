<template>
  <div v-if="childDataLoaded">
    <div style="margin-top: 30px">
      <button v-on:click="component = 'MC3LineChart'" id="MC3LineChart">Company overview</button>
      <button v-on:click="component = 'Network'" id="Network">Interactive Network</button>
    </div>
    <component v-bind:is='component' :linksArray="linksArray" :nodesArray="nodesArray"></component>
  </div>
</template>

<script>
import Network from '@/components/Network'
import MC3LineChart from '@/components/MC3LineChart'
// import * as d3 from 'd3'
export default {
  name: 'MC3',
  components: {
    MC3LineChart,
    Network
  },
  data () {
    return {
      component: 'MC3LineChart',
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
        this.childDataLoaded = true
      })
  }
}
</script>

<style scoped>

</style>
