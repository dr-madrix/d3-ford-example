<template>
<div class="bg-light py-1" id="bg-div">
  <FordHeader />
  <main-viz />
</div>
</template>

<script>
import FordHeader from "./components/FordHeader.vue"
import MainViz from './components/MainViz.vue'

export default {
  name: 'App',
  components: {
    FordHeader,
    MainViz
  }
}
</script>

<style lang="scss">
#bg-div{
  height: 100vh;
}
</style>
