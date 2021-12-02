<template>
  <div id="app">
    <Ver2 msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import Ver2 from './components/Ver2.vue'

export default {
  name: 'App',
  components: {
    Ver2
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
