<template>
  <div class="player">
    <VideoList :videoEl="videoEl" @updateSrc="updateSrc" />
    <div class="video-wrapper">
      <div class="border"></div>
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
      <VideoController
        v-if="videoEl !== null"
        :videoEl="videoEl"
        @resetVideo="handleResetVideo"
        @backwardVideo="handleBackwardVideo"
        @forwardVideo="handleForwardVideo"
        @endVideo="handleEndVideo"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import DrawCanvas from "./DrawCanvas.vue";
import VideoController from "./VideoController.vue";
import VideoList from "./VideoList.vue";

export default defineComponent({
  components: { VideoList, DrawCanvas, VideoController },
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
    const handleResetVideo = () => {
      if (videoEl.value) {
        videoEl.value.currentTime = 0;
      }
    };
    const handleBackwardVideo = (interval: number) => {
      if (videoEl.value) {
        videoEl.value.currentTime -= interval;
      }
    };
    const handleForwardVideo = (interval: number) => {
      if (videoEl.value) {
        videoEl.value.currentTime += interval;
      }
    };
    const handleEndVideo = () => {
      if (videoEl.value) {
        videoEl.value.currentTime = videoEl.value.duration;
      }
    };

    onMounted(() => {
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
    });
    return {
      videoEl,
      canvas,
      ctx,
      updateSrc,
      handleResetVideo,
      handleBackwardVideo,
      handleForwardVideo,
      handleEndVideo,
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
