<template>
  <div id="app">
    <panoramic-viewer />
  </div>
</template>

<script>
import PanoramicViewer from '/src/appComponents/PanoramView.vue';

export default {
  name: 'App',
  components: {
    PanoramicViewer,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
