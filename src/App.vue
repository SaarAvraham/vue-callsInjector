<template>
  <div id="app">
      <vue-headful
              title="Calls Injector"
      />
      <HelloWorld msg="Calls Injector"/>
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

    :root {
        --status-indicator-size: 15px;
        --status-indicator-animation-duration: 2s;

        --status-indicator-color: rgb(216, 226, 233);
        --status-indicator-color-semi: rgba(216, 226, 233, .5);
        --status-indicator-color-transparent: rgba(216, 226, 233, 0);

        --status-indicator-color-active: rgb(0, 149, 255);
        --status-indicator-color-active-semi: rgba(0, 149, 255, .5);
        --status-indicator-color-active-transparent: rgba(0, 149, 255, 0);

        --status-indicator-color-positive: rgb(75, 210, 143);
        --status-indicator-color-positive-semi: rgba(75, 210, 143, .5);
        --status-indicator-color-positive-transparent: rgba(75, 210, 143, 0);

        --status-indicator-color-intermediary: rgb(255, 170, 0);
        --status-indicator-color-intermediary-semi: rgba(255, 170, 0, .5);
        --status-indicator-color-intermediary-transparent: rgba(255, 170, 0, 0);

        --status-indicator-color-negative: rgb(255, 77, 77);
        --status-indicator-color-negative-semi: rgba(255, 77, 77, .5);
        --status-indicator-color-negative-transparent: rgba(255, 77, 77, 0);
    }
</style>
