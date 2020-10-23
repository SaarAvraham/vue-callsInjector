<template>
  <div id="app">
      <vue-headful
              title="Data Lake Calls Injector"
      />
      <HelloWorld msg="Data Lake Calls Injector"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import vueHeadful from 'vue-headful';

export default {
  name: 'App',
  components: {
    HelloWorld, vueHeadful
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
