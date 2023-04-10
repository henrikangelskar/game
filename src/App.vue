<template>
  <div id="game">
    <GameBoard />
  </div>
</template>
<script setup>
import GameBoard from "./components/GameBoard"
</script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

*{
  font-family: 'Press Start 2P', cursive !important;
}
html{
  padding: 0px;
  margin: 0px;
  background: black;
  color: white;
}
#game{
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>