<template>
  <div id="app">
    <MC2></MC2>
  </div>
</template>

<script>
import MC2 from './components/MC2'
import MC3 from './components/MC3'

export default {
  name: 'App',
  components: {
    MC2,
    MC3
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
