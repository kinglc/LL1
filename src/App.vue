<template>
  <div id="app">
    <ll1/>
  </div>
</template>

<script>
import ll1 from './components/ll1'

export default {
  name: 'App',
  components: {
    ll1
  }
}
</script>

<style>
  *{margin:0;padding:0}

  #app {
    position: absolute;
    width: 100%;
    height: 100%;
    padding: 0;
    overflow: hidden;
  }
</style>
