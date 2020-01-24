<template>
  <div v-if="childDataLoaded">
    <!--la proprietà che passi e come la chiami-->
    <div>
    <button v-on:click="component = 'MC2LineChartSniper'" id="MC2LineChartSniper">Line Chart</button>
      <button v-on:click="component = 'InteractiveDashboard'" id="InteractiveDashboard">Dashboard</button>
    </div>
    <keep-alive>
      <component v-bind:is='component' :readyArray="readyArray" :singleLocation="singleLocation" :singleMeasure="singleMeasure"></component>
    </keep-alive>
  </div>
</template>

<script>
/* eslint-disable */
  import MC2LineChartSniper from '@/components/MC2LineChartSniper'
import InteractiveDashboard from '@/components/InteractiveDashboard'


  import * as d3 from 'd3'
  export default {
    name: 'MC2',
    components: {
      'MC2LineChartSniper': MC2LineChartSniper,
      'InteractiveDashboard': InteractiveDashboard
    },
    data () {
      return {
        component: 'InteractiveDashboard',
        childDataLoaded: false,
        readyArray: [],
        measure: null,
        sampleDate: null,
        location: null,
        value: null,
        singleLocation: [],
        singleMeasure: [],
        unit: null
      }
    },

    mounted () {
      // let measure
      let parseTime = d3.timeParse('%Y-%m-%d')
      fetch('/static/data/d3DefinitivoSoluzioni.json')
        .then(res => res.json())
        .then((res) => {
          const measure = res.map((d) => {
            const r = {

              location: d.location,
              measure: d.measure,
              unit: d.unit,
              value: +d.value,
              year: +d.sampleDate.split('-')[0],
              sampleDate: parseTime(d.sampleDate)
            }
            return r
          })
          this.readyArray = measure
          this.childDataLoaded = true
          this.singleLocation = [...new Set(this.readyArray.map(item => item.location))]
          this.singleMeasure = [...new Set(this.readyArray.map(item => item.measure))]
        })
    }
  }
</script>

<style scoped>

</style>
