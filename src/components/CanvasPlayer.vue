<template>
  <div class="player">
    <div class="video-wrapper">
      <h1>{{ state.timeCode }} / {{ state.metaData.fps[state.mediaIndex] }}</h1>
      <button @click="changeMedia" aria-label="change media">
        Change Media
      </button>
      <div class="test">
        <video ref="mediaEl">
          <source src="" type="video/mp4" />
        </video>
        <canvas ref="canvas"></canvas>
      </div>
      <div class="controls" ref="controls">
        <button
          class="play"
          @click="togglePlay"
          data-icon="P"
          aria-label="play pause toggle"
        ></button>
        <button
          class="stop"
          @click="toggleStop"
          data-icon="S"
          aria-label="stop"
        ></button>
        <div class="timer" ref="timerWrapper" @click="seekToTime">
          <div ref="progressbar"></div>
          <span aria-label="timer"
            >{{ state.formattedTime }} / {{ state.mediaDuration }}</span
          >
        </div>
        <button
          class="rwd"
          @click="goBackward"
          data-icon="B"
          aria-label="rewind"
        ></button>
        <button
          class="fwd"
          @click="goForward"
          data-icon="F"
          aria-label="fast forward"
        ></button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, reactive, defineComponent, computed } from "vue";

interface State {
  mediaIndex: number;
  fileNames: string[];
  formattedTime: string;
  mediaDuration: string;
  intervalFwd: number;
  intervalRwd: number;
  isDragging: boolean;
  animationFrameId: number;
  skipTime: number;
  /* lastMediaTime: number;
  lastFrameNum: number;
  fpsRounder: number[];
  frameSeek: boolean;
  fps: number;*/
  timeCode: string;
  metaData: MetaData;
}
/* interface VideoMetadata {
  mediaTime: number;
  presentedFrames: number;
} */

interface MetaData {
  width: number[];
  height: number[];
  videoBitrate: number[];
  fps: number[];
  audioBitrate: number[];
  audioSample: number[];
}

export default defineComponent({
  components: {},
  setup() {
    const state = reactive<State>({
      mediaIndex: 0,
      fileNames: [],
      formattedTime: "00:00",
      mediaDuration: "00:00",
      intervalFwd: 0,
      intervalRwd: 0,
      isDragging: false,
      animationFrameId: 0,
      skipTime: 10,
      /* lastMediaTime: 0,
      lastFrameNum: 0,
      fpsRounder: [],
      frameSeek: true,
      fps: 0, */
      timeCode: "00:00:00:00",
      metaData: {
        width: [720, 1080],
        height: [306, 1918],
        videoBitrate: [1332, 1082],
        fps: [24, 23.98],
        audioBitrate: [149, 0],
        audioSample: [48, 0],
      },
    });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const mediaEl = ref<HTMLVideoElement | null>(null);
    const controls = ref<HTMLElement | null>(null);
    const timerWrapper = ref<HTMLElement | null>(null);
    const progressbar = ref<HTMLElement | null>(null);

    const smpteTimeCode = (currentTime: number) => {
      const mediaFps = state.metaData.fps[state.mediaIndex];
      const totalFrames = Math.floor(currentTime * mediaFps);
      const frames = totalFrames % mediaFps;

      const totalSeconds = Math.floor(totalFrames / mediaFps);
      const seconds = totalSeconds % 60;

      const totalMinutes = Math.floor(totalSeconds / 60);
      const minutes = totalMinutes % 60;

      const hours = Math.floor(totalMinutes / 60);
      return `${hours.toString().padStart(2, "0")}:${minutes
        .toString()
        .padStart(2, "0")}:${seconds.toString().padStart(2, "0")}:${frames
        .toString()
        .padStart(2, "0")}`;
    };

    /* const fpsDisplay = computed(() => {
      return `FPS: ${state.fps}`;
    });
    const getFpsAverage = () => {
      return (
        state.fpsRounder.reduce((a, b) => a + b, 0) / state.fpsRounder.length
      );
    };

    const ticker = (timeupdate: number, metadata: VideoMetadata) => {
      const mediaTimeDiff = Math.abs(metadata.mediaTime - state.lastMediaTime);
      const frameNumDiff = Math.abs(
        metadata.presentedFrames - state.lastFrameNum
      );
      const diff = mediaTimeDiff / frameNumDiff;
      if (
        diff &&
        diff < 1 &&
        state.frameSeek &&
        state.fpsRounder.length < 50 &&
        mediaEl.value?.playbackRate === 1 &&
        document.hasFocus()
      ) {
        state.fpsRounder.push(diff);
        state.fps = Math.round(1 / getFpsAverage());
      }
      state.frameSeek = true;
      state.lastMediaTime = metadata.mediaTime;
      state.lastFrameNum = metadata.presentedFrames;
      mediaEl.value?.requestVideoFrameCallback(ticker);
    };
    const handleSeek = () => {
      if (state.fpsRounder.length > 0) {
        state.fpsRounder.pop();
      }
      state.frameSeek = false;
    }; */

    const drawCanvas = () => {
      if (canvas.value && ctx.value && mediaEl.value) {
        const videoAspectRatio =
          mediaEl.value.videoWidth / mediaEl.value.videoHeight;
        const canvasAspectRatio = canvas.value.width / canvas.value.height;
        const { drawWidth, drawHeight, xStart, yStart } = (() => {
          if (videoAspectRatio < canvasAspectRatio) {
            const height = canvas.value.height;
            const width = height * videoAspectRatio;
            return {
              drawWidth: width,
              drawHeight: height,
              xStart: (canvas.value.width - width) / 2,
              yStart: 0,
            };
          } else {
            const width = canvas.value.width;
            const height = width / videoAspectRatio;
            return {
              drawWidth: width,
              drawHeight: height,
              xStart: 0,
              yStart: (canvas.value.height - height) / 2,
            };
          }
        })();
        ctx.value.drawImage(
          mediaEl.value,
          xStart,
          yStart,
          drawWidth,
          drawHeight
        );
        if (mediaEl.value.paused) {
          cancelAnimationFrame(state.animationFrameId);
        } else {
          state.animationFrameId = requestAnimationFrame(drawCanvas);
        }
      }
    };

    const togglePlay = () => {
      if (mediaEl.value) {
        clearIntervalRwdFwd();
        if (mediaEl.value.paused) {
          mediaEl.value.play();
          drawCanvas();
        } else {
          mediaEl.value.pause();
        }
      }
    };

    const toggleStop = () => {
      if (mediaEl.value) {
        clearIntervalRwdFwd();
        mediaEl.value.pause();
        mediaEl.value.currentTime = 0;
      }
    };

    const clearIntervalRwdFwd = () => {
      if (state.intervalRwd) {
        clearInterval(state.intervalRwd);
      }
      if (state.intervalFwd) {
        clearInterval(state.intervalFwd);
      }
    };

    const goBackward = () => {
      if (mediaEl.value) {
        if (mediaEl.value.currentTime <= state.skipTime) {
          mediaEl.value.currentTime = 0;
          toggleStop();
        } else {
          mediaEl.value.currentTime -= state.skipTime;
        }
        drawCanvas();
      }
    };

    const goForward = () => {
      if (mediaEl.value) {
        if (
          mediaEl.value.currentTime >=
          mediaEl.value.duration - state.skipTime
        ) {
          mediaEl.value.currentTime = mediaEl.value.duration;
        } else {
          mediaEl.value.currentTime += state.skipTime;
        }
        drawCanvas();
      }
    };

    const setTime = () => {
      if (mediaEl.value && timerWrapper.value && progressbar.value) {
        state.timeCode = smpteTimeCode(mediaEl.value.currentTime);
        const minutes = Math.floor(mediaEl.value.currentTime / 60);
        const seconds = Math.floor(mediaEl.value.currentTime - minutes * 60);
        const totalMinutes = Math.floor(mediaEl.value.duration / 60);
        const totalSeconds = Math.floor(mediaEl.value.duration % 60);

        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");
        const totalMinuteValue = totalMinutes.toString().padStart(2, "0");
        const totalSecondValue = totalSeconds.toString().padStart(2, "0");

        const mediaTime = `${minuteValue}:${secondValue}`;
        state.formattedTime = mediaTime;

        const mediaDuration = `${totalMinuteValue}:${totalSecondValue}`;
        state.mediaDuration = mediaDuration;

        const barLength =
          (mediaEl.value.currentTime / mediaEl.value.duration) *
            timerWrapper.value.clientWidth || 0;
        progressbar.value.style.width = barLength + "px";
      }
    };

    const seekToTime = (event: MouseEvent) => {
      if (mediaEl.value && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const clickX = event.clientX - barRect.left;
        const barWidth = barRect.width;
        const newTime = (clickX / barWidth) * mediaEl.value.duration;
        mediaEl.value.currentTime = newTime;
        drawCanvas();
      }
    };
    const startDrag = (event: MouseEvent) => {
      state.isDragging = true;
      seekToTime(event);
    };

    const duringDrag = (event: MouseEvent) => {
      if (state.isDragging) {
        seekToTime(event);
      }
    };

    const endDrag = () => {
      state.isDragging = false;
    };

    const changeMedia = () => {
      state.mediaIndex = (state.mediaIndex + 1) % state.fileNames.length;

      if (mediaEl.value) {
        mediaEl.value.src = state.fileNames[state.mediaIndex];
      }
    };

    onMounted(() => {
      const context = require.context("../assets", false, /\.(mp4|webm)$/);
      const filenames = context
        .keys()
        .map((key) => key.replace("./", "../assets/"));
      state.fileNames = filenames;
      if (mediaEl.value && state.fileNames.length > 0) {
        mediaEl.value.src = state.fileNames[state.mediaIndex];
      }
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
      timerWrapper.value?.addEventListener("mousedown", startDrag);
      timerWrapper.value?.addEventListener("mousemove", duringDrag);
      document.addEventListener("mouseup", endDrag);

      mediaEl.value?.addEventListener("loadedmetadata", () => {
        if (canvas.value && mediaEl.value && ctx.value) {
          const computedStyle = getComputedStyle(mediaEl.value);
          canvas.value.width = parseInt(computedStyle.width, 10);
          canvas.value.height = parseInt(computedStyle.height, 10);
          /* mediaEl.value.requestVideoFrameCallback(ticker); */
          mediaEl.value.play();
          setTime();
          mediaEl.value.addEventListener("timeupdate", () => {
            setTime();
            drawCanvas();
          });
        }
      });
    });

    return {
      state,
      mediaEl,
      controls,
      timerWrapper,
      progressbar,
      goForward,
      goBackward,
      togglePlay,
      toggleStop,
      seekToTime,
      changeMedia,
      canvas,
      ctx,

      /* fpsDisplay,
      handleSeek, */
    };
  },
});
</script>

<style scoped>
@font-face {
  font-family: "HeydingsControlsRegular";
  src: url("../assets/fonts/heydings_controls-webfont.eot");
  src: url("../assets/fonts/heydings_controls-webfont.eot?#iefix")
      format("embedded-opentype"),
    url("../assets/fonts/heydings_controls-webfont.woff") format("woff"),
    url("../assets/fonts/heydings_controls-webfont.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
}
input {
  margin: 0;
  padding: 0;
}

canvas {
  /* border: 1px solid black; */
  height: 50vh;
  width: 100vh;
}
video {
  height: 50vh;
  width: 100vh;
  display: none;
}

p {
  position: absolute;
  top: 310px;
}

.player {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.video-wrapper {
  position: relative;
}

.controls {
  opacity: 0;
  width: 400px;
  border-radius: 10px;
  position: absolute;
  bottom: -50px;
  left: 50%;
  background-color: black;
  box-shadow: 3px 3px 5px black;
  transition: 1s all;
  display: flex;
  z-index: 10;
  transform: translateX(-50%);
}

button,
.controls {
  background: linear-gradient(to bottom, #222, #666);
}

.video-wrapper:hover .controls,
.video-wrapper:focus .controls {
  opacity: 1;
}

button:before {
  font-family: HeydingsControlsRegular;
  font-size: 20px;
  position: relative;
  content: attr(data-icon);
  color: #aaa;
  text-shadow: 1px 1px 0px black;
}

.play:before {
  font-size: 22px;
}

button,
.timer {
  height: 38px;
  line-height: 19px;
  box-shadow: inset 0 -5px 25px rgba(0, 0, 0, 0.3);
  border-right: 1px solid #333;
}

button {
  position: relative;
  border: 0;
  flex: 1;
  outline: none;
  color: aqua;
}

.play {
  border-radius: 10px 0 0 10px;
}

.fwd {
  border-radius: 0 10px 10px 0;
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

button:hover,
button:focus {
  box-shadow: inset 1px 1px 2px black;
}

button:active {
  box-shadow: inset 3px 3px 2px black;
}
</style>
