<template>
  <ComponentController msg="ComponentController" />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ComponentController from "./components/ComponentController.vue";
export default defineComponent({
  name: "App",
  components: {
    ComponentController,
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
body {
  margin: 0;
}
</style>
