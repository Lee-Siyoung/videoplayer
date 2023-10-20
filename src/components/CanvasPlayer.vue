<template>
  <div class="player">
    <div>
      <li v-for="video in state.IVideo" :key="video.name">{{ video.name }}</li>
    </div>
    <div class="video-wrapper">
      <button @click="changeMedia">Change Media</button>
      <div class="test">
        <video ref="mediaEl">
          <source src="" type="video/mp4" />
        </video>
        <canvas ref="canvas"></canvas>
      </div>
      <div class="controls" ref="controlEl">
        <button class="play" @click="togglePlay" data-icon="P"></button>
        <button class="stop" @click="toggleStop" data-icon="S"></button>
        <button class="rwd" @click="goBackward" data-icon="B"></button>
        <button class="fwd" @click="goForward" data-icon="F"></button>
      </div>
      <div class="timer" ref="timerWrapper" @click="seekToTime">
        <div ref="progressbar"></div>
        <span>{{ state.timeCode }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, reactive, defineComponent } from "vue";

interface State {
  mediaIndex: number;
  formattedTime: string;
  mediaDuration: string;
  intervalFwd: number;
  intervalRwd: number;
  isDragging: boolean;
  animationFrameId: number;
  skipTime: number;
  timeCode: string;
  IVideo: IVideo[];
}

interface IVideo {
  src: string;
  name: string;
  fps: number;
}

export default defineComponent({
  components: {},
  setup() {
    const state = reactive<State>({
      mediaIndex: 0,
      formattedTime: "00:00",
      mediaDuration: "00:00",
      intervalFwd: 0,
      intervalRwd: 0,
      isDragging: false,
      animationFrameId: 0,
      skipTime: 10,
      timeCode: "00:00:00:00",
      IVideo: [
        {
          src: "",
          name: "",
          fps: 24,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
        },
      ],
    });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const mediaEl = ref<HTMLVideoElement | null>(null);
    const controlEl = ref<HTMLElement | null>(null);
    const timerWrapper = ref<HTMLElement | null>(null);
    const progressbar = ref<HTMLElement | null>(null);

    const smpteTimeCode = (currentTime: number) => {
      const mediaFps = state.IVideo[state.mediaIndex].fps;
      const totalFrames = Math.floor(currentTime * mediaFps);
      const hours = Math.floor(totalFrames / (60 * 60 * mediaFps));
      const minutes = Math.floor((totalFrames / (60 * mediaFps)) % 60);
      const seconds = Math.floor((totalFrames / mediaFps) % 60);
      const frames = Math.floor(totalFrames % mediaFps);
      return `${hours.toString().padStart(2, "0")}:${minutes
        .toString()
        .padStart(2, "0")}:${seconds.toString().padStart(2, "0")}:${frames
        .toString()
        .padStart(2, "0")}`;
    };
    const update = () => {
      if (mediaEl.value) {
        state.timeCode = smpteTimeCode(mediaEl.value.currentTime);
      }
    };

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
      state.mediaIndex = (state.mediaIndex + 1) % state.IVideo.length;

      if (mediaEl.value) {
        mediaEl.value.src = state.IVideo[state.mediaIndex].src;
      }
    };

    onMounted(() => {
      const context = require.context("../assets", false, /\.(mp4|webm)$/);
      const filenames = context
        .keys()
        .map((key) => key.replace("./", "").replace(".mp4", ""));
      const url = context.keys().map((key) => key.replace("./", "../assets/"));
      let index = 0;
      for (const key in state.IVideo) {
        state.IVideo[key].src = url[index];
        state.IVideo[key].name = filenames[index];
        index++;
      }
      if (mediaEl.value) {
        mediaEl.value.src = state.IVideo[state.mediaIndex].src;
      }
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
      timerWrapper.value?.addEventListener("mousedown", startDrag);
      timerWrapper.value?.addEventListener("mousemove", duringDrag);
      timerWrapper.value?.addEventListener("mouseup", endDrag);

      mediaEl.value?.addEventListener("loadedmetadata", () => {
        if (canvas.value && mediaEl.value && ctx.value) {
          const computedStyle = getComputedStyle(mediaEl.value);
          canvas.value.width = parseInt(computedStyle.width, 10);
          canvas.value.height = parseInt(computedStyle.height, 10);
          mediaEl.value.play();
          setTime();
          setInterval(update, 1000 / 24);
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
      controlEl,
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

canvas {
  height: 50vh;
  width: 100vh;
}
video {
  height: 50vh;
  width: 100vh;
  display: none;
}

.player {
  display: flex;
  position: relative;
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
  line-height: 70px;
  color: #0000000d;
  height: 5vw;
  position: relative;
}

.timer div {
  position: absolute;
  background-color: rgba(120, 120, 120, 0.233);
  left: 0;
  top: 0;
  width: 0;
  height: 5vw;
  z-index: 2;
}

.timer span {
  font-size: 15px;
  font-family: monospace;
  text-shadow: 1px 1px 0px black;
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
