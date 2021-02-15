<!-- This file will be used to layout and connect your views -->
<template>
  <div id="app">
      <!-- This is only a basic setup to display your views. Use HTML and CSS to create your layout using these basic commands -->
      <View3 />
  </div>
</template>

<script>

import View3 from './components/View3.vue'

export default {
  name: 'App',
  components: {
    View3
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
