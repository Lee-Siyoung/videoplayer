<template>
  <div class="player">
    <VideoList
      :videoEl="videoEl"
      @updateSrc="updateSrc"
      @updateState="updateState"
    />
    <div class="video-wrapper">
      <div class="border"></div>
      <h1>{{ state.name }}</h1>
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
        :autoPlay="state.autoPlay"
        @resetVideo="handleResetVideo"
        @backwardVideo="handleBackwardVideo"
        @forwardVideo="handleForwardVideo"
        @endVideo="handleEndVideo"
      />
      <ProgressBar
        v-if="videoEl !== null"
        :videoEl="videoEl"
        :videoData="videoData"
        @seekVideo="handleSeekVideo"
      />
    </div>
    <VideoSetting
      v-if="videoEl !== null"
      :videoEl="videoEl"
      @updateAutoPlay="handleUpdateAutoPlay"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, reactive } from "vue";
import DrawCanvas from "./DrawCanvas.vue";
import ProgressBar from "./ProgressBar.vue";
import VideoController from "./VideoController.vue";
import VideoList from "./VideoList.vue";
import VideoSetting from "./VideoSetting.vue";
interface State {
  autoPlay: boolean;
  name: string;
}
interface VideoState {
  videoIndex: number;
  IVideo: IVideo[];
}
interface IVideo {
  src: string;
  name: string;
  fps: number;
}
export default defineComponent({
  components: {
    VideoList,
    DrawCanvas,
    VideoController,
    ProgressBar,
    VideoSetting,
  },

  setup() {
    const videoEl = ref<HTMLVideoElement | null>(null);
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const videoData = ref<VideoState>({
      videoIndex: 0,
      IVideo: [],
    });
    const state = reactive<State>({ autoPlay: false, name: "" });
    const updateSrc = (src: string) => {
      if (videoEl.value) {
        videoEl.value.src = src;
      }
    };

    const updateState = (updatedState: VideoState) => {
      videoData.value = updatedState;
      state.name = videoData.value.IVideo[videoData.value.videoIndex].name;
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
    const handleUpdateAutoPlay = (autoPlay: boolean) => {
      state.autoPlay = autoPlay;
    };

    onMounted(() => {
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
    });
    return {
      state,
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
      handleUpdateAutoPlay,
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
h1 {
  color: #fff;
  padding: 10px;
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
