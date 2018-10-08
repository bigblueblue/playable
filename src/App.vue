<template>
  <div id="app">
    <!-- <merge-plane></merge-plane> -->
    <marquee-plane></marquee-plane>
    <!-- <phaser-test></phaser-test> -->
    <!-- <scratch-play></scratch-play> -->
  </div>
</template>

<script>
// import mergePlane from './components/mergePlane'
import marqueePlane from './components/marqee_plane'
// import phaserTest from './components/learning/phaser-plane'
// import scratchPlay from './components/scratch_play'
export default {
  name: 'App',
  components: {
    // mergePlane,
    marqueePlane
    // phaserTest
    // scratchPlay
  }
}
</script>

<style>
#app, body, html {
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    width: 100%;
    height: 100%;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    overflow: hidden;
}
</style>
