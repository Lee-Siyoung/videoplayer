<template>
  <div class="player">
    <VideoList />
    <div class="video-wrapper"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import VideoList from "./VideoList.vue";

export default defineComponent({
  components: { VideoList },
  setup() {
    return {};
  },
});
</script>

<style scoped>
.player {
  display: flex;
  position: relative;
  height: 100vh;
  background: #27282c;
}
</style>
