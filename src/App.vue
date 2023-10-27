<template>
  <CanvasPlayer msg="CanvasPlayer" />
</template>

<script lang="ts">
import { defineComponent } from "vue";
/* import CanvasPlayer from "./components/CanvasPlayer.vue"; */
import CanvasPlayer from "./components/ValueControler.vue";
export default defineComponent({
  name: "App",
  components: {
    CanvasPlayer,
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
}
</style>
