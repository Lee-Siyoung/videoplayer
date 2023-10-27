<template>
  <div class="player">
    <VideoList :videoEl="videoEl" @updateVideoSrc="updateVideoSrc" />
  </div>
</template>

<script lang="ts">
import { ref, defineComponent } from "vue";
import VideoList from "./VideoList.vue";

export default defineComponent({
  components: { VideoList },
  setup() {
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const videoEl = ref<HTMLVideoElement | null>(null);
    const updateVideoSrc = (newSrc: string) => {
      if (videoEl.value) {
        videoEl.value.src = newSrc;
      }
    };

    return {
      videoEl,
      updateVideoSrc,
    };
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
