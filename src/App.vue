<template>
  <div id="app">
    <h1>Let's play Vue Gitman.</h1>
    <GameBoard o/>
  </div>
</template>

<script>
import GameBoard from "./components/GameBoard.vue";

export default {
  name: "app",
  components: {
    GameBoard
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=IBM+Plex+Mono");
#app {
  font-family: "IBM Plex Mono", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
