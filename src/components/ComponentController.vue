<template>
  <div class="player">
    <VideoList
      :videoEl="videoEl"
      @updateSrc="updateSrc"
      @updateState="updateState"
    />
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
      <ProgressBar
        v-show="videoEl !== null"
        :videoEl="videoEl"
        :videoData="videoData"
        @seekVideo="handleSeekVideo"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import DrawCanvas from "./DrawCanvas.vue";
import ProgressBar from "./ProgressBar.vue";
import VideoController from "./VideoController.vue";
import VideoList from "./VideoList.vue";

export default defineComponent({
  components: { VideoList, DrawCanvas, VideoController, ProgressBar },
  setup() {
    const videoEl = ref<HTMLVideoElement | null>(null);
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const videoData = ref({
      videoIndex: 0,
      IVideo: [],
    });
    const updateSrc = (src: string) => {
      if (videoEl.value) {
        videoEl.value.src = src;
      }
    };
    const updateState = (updatedState: any) => {
      videoData.value = updatedState;
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
    const handleSeekVideo = (time: number) => {
      if (videoEl.value) {
        videoEl.value.currentTime = time;
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
      handleSeekVideo,
      videoData,
      updateState,
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
.video-wrapper {
  position: relative;
  left: 10vw;
}
.border {
  position: absolute;
  right: -54px;
  top: 1vw;
  border: 1px solid #fff;
  width: 55vw;
  height: 45vw;
}
video {
  height: 50vh;
  width: 100vh;
  display: none;
}
canvas {
  height: 50vh;
  width: 100vh;
}
</style>
