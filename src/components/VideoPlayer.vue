<template>
  <div class="player">
    <div class="video-wrapper">
      <h1>Movie</h1>
      <button @click="changeMedia" aria-label="change media">
        Change Media
      </button>
      <button @click="toggleTextEdit" aria-label="Text Edit">Text Edit</button>
      <VideoCapture :canvas="canvas" :mediaEl="mediaEl" :ctx="ctx" />
      <div class="test">
        <video ref="mediaEl">
          <source src="" type="video/mp4" />
        </video>
        <canvas
          ref="canvas"
          @click="onCanvasClick"
          v-bind:style="{ 'pointer-events': state.isEditing ? 'auto' : 'none' }"
        ></canvas>
        <input
          v-if="state.hasInput"
          ref="textInput"
          type="text"
          v-bind:style="inputStyle"
          @keydown.enter="handleEnter"
        />
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
          @click="stopMedia"
          data-icon="S"
          aria-label="stop"
        ></button>
        <div
          class="timer"
          ref="timerWrapper"
          @click="seekToTime"
          @mousemove="showTimeOnHover"
          @mouseleave="hideTimeOnLeave"
        >
          <div ref="progressbar"></div>
          <span aria-label="timer">{{ state.formattedTime }}</span>
        </div>
        <button
          class="rwd"
          @click="Backward"
          data-icon="B"
          aria-label="rewind"
        ></button>
        <button
          class="fwd"
          @click="Forward"
          data-icon="F"
          aria-label="fast forward"
        ></button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, nextTick, reactive } from "vue";
import VideoCapture from "./VideoCapture.vue";

interface InputStyle {
  position: string;
  left: string;
  top: string;
}

interface State {
  isEditing: boolean;
  hasInput: boolean;
  mediaIndex: number;
  fileNames: string[];
  isPlaying: boolean;
  formattedTime: string;
  intervalFwd: number;
  intervalRwd: number;
}

export default {
  components: { VideoCapture },
  setup() {
    const state = reactive<State>({
      isEditing: true,
      hasInput: false,
      mediaIndex: 0,
      fileNames: [],
      isPlaying: false,
      formattedTime: "00:00",
      intervalFwd: 0,
      intervalRwd: 0,
    });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const textInput = ref<HTMLInputElement | null>(null);
    const inputStyle = ref<InputStyle>({
      position: "absolute",
      left: "0px",
      top: "0px",
    });

    const mediaEl = ref<HTMLVideoElement | null>(null);
    const controls = ref<HTMLElement | null>(null);
    const timerWrapper = ref<HTMLElement | null>(null);
    const progressbar = ref<HTMLElement | null>(null);

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

        requestAnimationFrame(drawCanvas);
      }
    };

    const toggleTextEdit = () => {
      state.isEditing = !state.isEditing;
    };

    const onCanvasClick = (event: MouseEvent) => {
      const targetElement = event.target as HTMLElement;
      const rect = targetElement.getBoundingClientRect();
      if (state.isEditing && !state.hasInput) {
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        console.log("클릭 x,y 위치 : ", x, y);
        addInput(x, y);
      }
    };

    const addInput = (x: number, y: number) => {
      inputStyle.value = {
        position: "absolute",
        left: `${x}px`,
        top: `${y + 110}px`,
      };
      console.log("input x,y 위치 : ", inputStyle.value);
      state.hasInput = true;

      nextTick(() => {
        textInput.value?.focus();
      });
    };
    const handleEnter = () => {
      if (textInput.value) {
        const x = parseInt(textInput.value.style.left, 10);
        const y = parseInt(textInput.value.style.top, 10);
        drawText(textInput.value.value, x, y - 110);
        state.hasInput = false;
        console.log("엔터 x,y 위치 : ", x, y);
      }
    };
    const drawText = (txt: string, x: number, y: number) => {
      if (ctx.value) {
        ctx.value.textBaseline = "top";
        ctx.value.font = "14px sans-serif";
        ctx.value.fillText(txt, x, y);
      }
    };

    const togglePlay = () => {
      if (mediaEl.value) {
        clearIntervalRwdFwd();
        if (mediaEl.value.paused) {
          mediaEl.value.play();
          state.isPlaying = true;
          drawCanvas();
        } else {
          mediaEl.value.pause();
          state.isPlaying = false;
        }
      }
    };

    const stopMedia = () => {
      if (mediaEl.value) {
        clearIntervalRwdFwd();
        mediaEl.value.pause();
        mediaEl.value.currentTime = 0;
        state.isPlaying = false;
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

    const Backward = () => {
      if (mediaEl.value) {
        if (mediaEl.value.currentTime <= 10) {
          stopMedia();
        } else {
          mediaEl.value.currentTime -= 10;
        }
      }
    };

    const Forward = () => {
      if (mediaEl.value) {
        if (mediaEl.value.currentTime >= mediaEl.value.duration - 10) {
          stopMedia();
        } else {
          mediaEl.value.currentTime += 10;
        }
      }
    };

    const setTime = () => {
      if (mediaEl.value && timerWrapper.value && progressbar.value) {
        const minutes = Math.floor(mediaEl.value.currentTime / 60);
        const seconds = Math.floor(mediaEl.value.currentTime - minutes * 60);

        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");

        const mediaTime = `${minuteValue}:${secondValue}`;
        state.formattedTime = mediaTime;

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
      }
    };

    const showTimeOnHover = (event: MouseEvent) => {
      if (mediaEl.value && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const hoverX = event.clientX - barRect.left;
        const barWidth = barRect.width;

        const hoverTime = (hoverX / barWidth) * mediaEl.value.duration;
        const minutes = Math.floor(hoverTime / 60);
        const seconds = Math.floor(hoverTime - minutes * 60);
        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");
        state.formattedTime = `${minuteValue}:${secondValue}`;
      }
    };

    const hideTimeOnLeave = () => {
      if (mediaEl.value) {
        setTime();
      }
    };

    const changeMedia = () => {
      state.mediaIndex = (state.mediaIndex + 1) % state.fileNames.length;

      if (mediaEl.value) {
        mediaEl.value.src = state.fileNames[state.mediaIndex];
        mediaEl.value.load();
        mediaEl.value.play();
      }
      console.log(state.mediaIndex);
    };

    onMounted(() => {
      const context = require.context("../assets", false, /\.(mp4|webm)$/);
      const filenames = context
        .keys()
        .map((key) => key.replace("./", "../assets/"));
      state.fileNames = filenames;
      if (mediaEl.value && state.fileNames.length > 0) {
        mediaEl.value.src = state.fileNames[state.mediaIndex];
        mediaEl.value.load();
      }

      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }

      mediaEl.value?.addEventListener("loadedmetadata", () => {
        if (canvas.value && mediaEl.value && ctx.value) {
          const computedStyle = getComputedStyle(mediaEl.value);
          canvas.value.width = parseInt(computedStyle.width, 10);
          canvas.value.height = parseInt(computedStyle.height, 10);
          mediaEl.value.addEventListener("timeupdate", setTime);
          drawCanvas();
          mediaEl.value.play();
        }
      });
    });

    return {
      state,
      mediaEl,
      controls,
      timerWrapper,
      progressbar,
      Forward,
      Backward,
      togglePlay,
      stopMedia,
      seekToTime,
      showTimeOnHover,
      hideTimeOnLeave,
      changeMedia,
      canvas,
      ctx,
      textInput,
      inputStyle,
      onCanvasClick,
      handleEnter,
      toggleTextEdit,
    };
  },
};
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
  bottom: 10px;
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
