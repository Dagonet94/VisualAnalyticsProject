<template>
  <div id="app">
    <b-navbar toggleable="lg" type="dark" variant="info" align="center">
      <b-navbar-brand href="#">Vast Challenge 2018</b-navbar-brand>

      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item v-on:click="comp = 'MC2'">Mini challenge 2</b-nav-item>
          <b-nav-item v-on:click="comp = 'MC3'">Mini challenge 3</b-nav-item>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <keep-alive>
    <component v-bind:is='comp'></component>
    </keep-alive>
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
  },
  data () {
    return {
      comp: 'MC2'
    }
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
  margin-top: 0px;
}
</style>
