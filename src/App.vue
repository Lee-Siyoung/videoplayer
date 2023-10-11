<template>
  <VideoPlayer msg="VideoPlayer" />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import VideoPlayer from "./components/VideoPlayer.vue";

export default defineComponent({
  name: "App",
  components: {
    VideoPlayer,
  },
});
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
