<template>
  <div class="player">
    <VideoList :videoEl="videoEl" @updateSrc="updateSrc" />
    <div class="video-wrapper">
      <div class="test">
        <video ref="videoEl" controls>
          <source src="" type="video/mp4" />
        </video>
        <canvas ref="canvas"></canvas>
        <DrawCanvas
          v-if="videoEl !== null"
          :videoEl="videoEl"
          :canvas="canvas"
          :ctx="ctx"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import DrawCanvas from "./DrawCanvas.vue";
import VideoList from "./VideoList.vue";

export default defineComponent({
  components: { VideoList, DrawCanvas },
  setup() {
    const videoEl = ref<HTMLVideoElement | null>(null);
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    console.log("controler");
    const updateSrc = (src: string) => {
      if (videoEl.value) {
        videoEl.value.src = src;
      }
    };
    onMounted(() => {
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
    });
    return { videoEl, canvas, ctx, updateSrc };
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
video {
  height: 50vh;
  width: 100vh;
  /* display: none; */
}
canvas {
  height: 50vh;
  width: 100vh;
}
</style>
