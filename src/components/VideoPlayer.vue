<template>
  <div class="player">
    <h1>VideoPlayer</h1>
    <video controls>
      <source src="../assets/sintel-short.mp4" type="video/mp4" />
      <source src="../assets/sintel-short.webm" type="video/webm" />
    </video>
    <div class="controls">
      <button
        class="play"
        data-icon="P"
        aria-label="play pause toggle"
      ></button>
      <button class="stop" data-icon="S" aria-label="stop"></button>
      <div class="timer">
        <span aria-label="timer">00:00</span>
      </div>
      <div></div>
      <button class="rwd" data-icon="B" aria-label="rewind"></button>
      <button class="fwd" data-icon="F" aria-label="fast forward"></button>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";

export default {
  name: "VideoPlayer",
  props: {
    msg: String,
  },
};
</script>

<style scoped>
h1 {
  margin: 40px 0 0;
}
.controls {
  /* visibility: hidden; */
  opacity: 0.5;
  width: 400px;
  border-radius: 10px;
  position: absolute;
  bottom: 20px;
  left: 50%;
  margin-left: -200px;
  background-color: black;
  box-shadow: 3px 3px 5px black;
  transition: 1s all;
  display: flex;
}

.player:hover .controls,
.player:focus-within .controls {
  opacity: 1;
}

@font-face {
  font-family: "HeydingsControlsRegular";
  src: url("../assets/fonts/heydings_controls-webfont.eot?iefix")
      format("embedded-opentype"),
    url("../assets/fonts/heydings_controls-webfont.woff")
      format("embedded-opentype"),
    url("../assets/fonts/heydings_controls-webfont.ttf")
      format("embedded-opentype");
  font-weight: normal;
  font-style: normal;
}

button::before {
  font-family: HeydingsControlsRegular;
  font-size: 20px;
  position: relative;
  content: attr(data-icon);
  color: #aaa;
  text-shadow: 1px 1px 0px black;
}

.timer {
  line-height: 38px;
  font-size: 10px;
  font-family: monospace;
  text-shadow: 1px 1px 0px black;
  color: white;
  flex: 5;
  position: relative;
}

.timer div {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.2);
  left: 0;
  top: 0;
  width: 0;
  height: 38px;
  z-index: 2;
}

.timer span {
  position: absolute;
  z-index: 3;
  left: 19px;
}
</style>
