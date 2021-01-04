<template>
    <div id="app">
        <sliding-puzzle></sliding-puzzle>
    </div>
</template>

<script>
import SlidingPuzzle from './components/SlidingPuzzle.vue'

export default {
    name: 'App',
    components: {
        SlidingPuzzle
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
