<template>
  <div v-if="childDataLoaded">
    <int-dash :dataArray="dataArray"></int-dash>
  </div>
</template>

<script>
import InteractiveDashboard from '@/components/InteractiveDashboard'

export default {
  name: 'MC2',
  components: {
    'int-dash': InteractiveDashboard
  },
  data () {
    return {
      childDataLoaded: false,
      location: '',
      measure: '',
      date: '',
      year: '',
      dataArray: []
    }
  },
  mounted () {
    fetch('/static/data/ds.json')
      .then(res => res.json())
      .then((res) => {
        const measurements = res.map((d) => {
          const r = {
            location: d.location,
            measure: d.measure,
            date: d.sampleDate,
            year: +d.sampleDate.split('-')[0]
          }
          return r
        })
        this.dataArray = measurements
        this.childDataLoaded = true
      })
  }
}
</script>

<style>

</style>
