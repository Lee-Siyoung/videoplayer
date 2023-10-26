<template>
  <div class="player">
    <div class="scroll">
      <li
        class="videoList"
        v-for="(video, index) in state.IVideo"
        :key="video.src"
      >
        <a
          class="video-button"
          @click="clickVideo(index)"
          :class="{ active: state.videoIndex === index }"
        >
          <span>
            {{ video.name }}
          </span>
          <i></i>
        </a>
      </li>
    </div>
    <div class="video-wrapper">
      <div class="border"></div>
      <h1>{{ state.IVideo[state.videoIndex].name }}</h1>
      <div class="test">
        <video ref="videoEl">
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
        <span class="time"
          >{{ state.formattedTime }} / {{ state.videoDuration }}</span
        >
      </div>
    </div>
    <div class="action">
      <div class="profile" @click="menuToggle">
        <img src="../assets/setting.png" />
      </div>
      <div class="radio-buttons" :class="{ menuActive: state.isActive }">
        <p>시작 설정</p>
        <label>
          <input
            type="radio"
            id="play"
            name="videoControl"
            value="true"
            @change="state.autoPlay = true"
          />
          <span class="check"></span>시작
        </label>
        <label>
          <input
            type="radio"
            id="stop"
            name="videoControl"
            value="false"
            @change="state.autoPlay = false"
            checked
          />
          <span class="check"></span>멈춤</label
        >
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, reactive, defineComponent } from "vue";

interface State {
  isActive: boolean;
  formattedTime: string;
  videoDuration: string;
  autoPlay: boolean;
  videoIndex: number;
  isDragging: boolean;
  animationFrameId: number;
  interval: number;
  timeCode: string;
  IVideo: IVideo[];
}

interface IVideo {
  src: string;
  name: string;
  fps: number;
}

export default defineComponent({
  setup() {
    const state = reactive<State>({
      isActive: false,
      formattedTime: "00:00",
      videoDuration: "00:00",
      autoPlay: false,
      videoIndex: 0,
      isDragging: false,
      animationFrameId: 0,
      interval: 10,
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
          fps: 25,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
        },
        {
          src: "",
          name: "",
          fps: 24,
        },
      ],
    });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const videoEl = ref<HTMLVideoElement | null>(null);
    const controlEl = ref<HTMLElement | null>(null);
    const timerWrapper = ref<HTMLElement | null>(null);
    const progressbar = ref<HTMLElement | null>(null);

    const menuToggle = () => {
      state.isActive = !state.isActive;
    };

    const smpteTimeCode = (currentTime: number) => {
      const videoFps = state.IVideo[state.videoIndex].fps;
      const totalFrames = Math.floor(currentTime * videoFps);
      const hours = Math.floor(totalFrames / (60 * 60 * videoFps));
      const minutes = Math.floor((totalFrames / (60 * videoFps)) % 60);
      const seconds = Math.floor((totalFrames / videoFps) % 60);
      const frames = Math.floor(totalFrames % videoFps);
      return `${hours.toString().padStart(2, "0")}:${minutes
        .toString()
        .padStart(2, "0")}:${seconds.toString().padStart(2, "0")}:${frames
        .toString()
        .padStart(2, "0")}`;
    };
    const update = () => {
      if (videoEl.value) {
        state.timeCode = smpteTimeCode(videoEl.value.currentTime);
      }
    };

    const drawCanvas = () => {
      if (canvas.value && ctx.value && videoEl.value) {
        const videoAspectRatio =
          videoEl.value.videoWidth / videoEl.value.videoHeight;
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
          videoEl.value,
          xStart,
          yStart,
          drawWidth,
          drawHeight
        );
        if (videoEl.value.paused) {
          cancelAnimationFrame(state.animationFrameId);
        } else {
          state.animationFrameId = requestAnimationFrame(drawCanvas);
        }
      }
    };

    const togglePlay = () => {
      if (videoEl.value) {
        if (videoEl.value.paused) {
          videoEl.value.play();
          drawCanvas();
        } else {
          videoEl.value.pause();
        }
      }
    };

    const toggleStop = () => {
      if (videoEl.value) {
        videoEl.value.pause();
        videoEl.value.currentTime = 0;
      }
    };

    const goBackward = () => {
      if (videoEl.value) {
        if (videoEl.value.currentTime <= state.interval) {
          videoEl.value.currentTime = 0;
          toggleStop();
        } else {
          videoEl.value.currentTime -= state.interval;
        }
        drawCanvas();
      }
    };

    const goForward = () => {
      if (videoEl.value) {
        if (
          videoEl.value.currentTime >=
          videoEl.value.duration - state.interval
        ) {
          videoEl.value.currentTime = videoEl.value.duration;
        } else {
          videoEl.value.currentTime += state.interval;
        }
        drawCanvas();
      }
    };

    const setTime = () => {
      if (videoEl.value && timerWrapper.value && progressbar.value) {
        state.timeCode = smpteTimeCode(videoEl.value.currentTime);
        const minutes = Math.floor(videoEl.value.currentTime / 60);
        const seconds = Math.floor(videoEl.value.currentTime - minutes * 60);
        const totalMinutes = Math.floor(videoEl.value.duration / 60);
        const totalSeconds = Math.floor(videoEl.value.duration % 60);

        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");
        const totalMinuteValue = totalMinutes.toString().padStart(2, "0");
        const totalSecondValue = totalSeconds.toString().padStart(2, "0");

        const videoTime = `${minuteValue}:${secondValue}`;
        state.formattedTime = videoTime;

        const videoDuration = `${totalMinuteValue}:${totalSecondValue}`;
        state.videoDuration = videoDuration;

        const barLength =
          (videoEl.value.currentTime / videoEl.value.duration) *
            timerWrapper.value.clientWidth || 0;
        progressbar.value.style.width = barLength + "px";
      }
    };

    const seekToTime = (event: MouseEvent) => {
      if (videoEl.value && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const clickX = event.clientX - barRect.left;
        const barWidth = barRect.width;
        const newTime = (clickX / barWidth) * videoEl.value.duration;
        videoEl.value.currentTime = newTime;
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

    const clickVideo = (index: number) => {
      state.videoIndex = index;
      if (videoEl.value) {
        videoEl.value.src = state.IVideo[state.videoIndex].src;
      }
    };

    onMounted(() => {
      const context = require.context("../assets", false, /\.(mp4|webm)$/);
      const filenames = context
        .keys()
        .map((key) => key.replace("./", "").replace(".mp4", ""));
      const url = context.keys().map((key) => key.replace("./", "../assets/"));
      let index = 0;
      const reg = /[-_/\\^$*+?.()|[\]{}]/g;
      for (const key in state.IVideo) {
        state.IVideo[key].src = url[index];
        state.IVideo[key].name = filenames[index]
          .replace(reg, " ")
          .toUpperCase();
        index++;
      }
      console.log(state.IVideo);
      if (videoEl.value) {
        videoEl.value.src = state.IVideo[state.videoIndex].src;
      }
      if (canvas.value) {
        ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
      }
      timerWrapper.value?.addEventListener("mousedown", startDrag);
      timerWrapper.value?.addEventListener("mousemove", duringDrag);
      timerWrapper.value?.addEventListener("mouseup", endDrag);
      videoEl.value?.addEventListener("loadedmetadata", () => {
        if (canvas.value && videoEl.value && ctx.value) {
          const computedStyle = getComputedStyle(videoEl.value);
          canvas.value.width = parseInt(computedStyle.width, 10);
          canvas.value.height = parseInt(computedStyle.height, 10);
          setTime();
          if (state.autoPlay) {
            togglePlay();
          } else {
            toggleStop();
          }
          setInterval(update, 1000 / 24);
          videoEl.value.addEventListener("timeupdate", () => {
            setTime();
            drawCanvas();
          });
        }
      });
    });

    return {
      state,
      videoEl,
      controlEl,
      timerWrapper,
      progressbar,
      menuToggle,
      goForward,
      goBackward,
      togglePlay,
      toggleStop,
      seekToTime,
      canvas,
      ctx,
      clickVideo,
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

.player {
  display: flex;
  position: relative;
  height: 100vh;
  background: #27282c;
}
h1 {
  color: #fff;
}
.videoList {
  flex-direction: column;
  gap: 50px;
  display: flex;
  align-items: left;
}
.video-button {
  position: relative;
  background: #fff;
  color: #fff;
  text-decoration: none;
  text-transform: uppercase;
  font-size: 1em;
  letter-spacing: 0.1em;
  font-weight: 400;
  padding: 10px 30px;
  transition: 0.5s;
}
.video-button:hover {
  background: #1e9bff;
  color: #1e9bff;
  box-shadow: 0 0 35px #1e9bff;
  letter-spacing: 0.25em;
}
.video-button:before {
  content: "";
  position: absolute;
  inset: 2px;
  background: #27282c;
}
.video-button span {
  position: relative;
  z-index: 1;
}
.video-button i {
  position: absolute;
  inset: 0;
  display: block;
}
.video-button i::before {
  content: "";
  position: absolute;
  top: 0;
  left: 80%;
  width: 10px;
  height: 4px;
  background: #27282c;
  transform: translateX(-50%) skewX(325deg);
  transition: 0.5s;
}
.video-button:hover i::before {
  width: 20px;
  left: 20%;
}
.video-button i::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 20%;
  width: 10px;
  height: 4px;
  background: #27282c;
  transform: translateX(-50%) skewX(325deg);
  transition: 0.5s;
}
.video-button:hover i::after {
  width: 20px;
  left: 80%;
}

.active,
.active:hover {
  background: #1e9bff;
  color: #1e9bff;
  box-shadow: 0 0 35px #1e9bff;
  letter-spacing: 0.25em;
  pointer-events: none;
}
.active i::before {
  width: 20px;
  left: 20%;
}
.active i::after {
  width: 20px;
  left: 80%;
}

.scroll {
  overflow: auto;
  height: 25.5vw;
}
/*  */
.action {
  position: relative;
  top: 20px;
  left: 400px;
  height: 30vw;
}
.action .profile {
  position: relative;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  overflow: hidden;
  cursor: pointer;
}
.action .profile img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: invert(99%) sepia(0%) saturate(3993%) hue-rotate(209deg)
    brightness(117%) contrast(100%);
}
.action .radio-buttons {
  position: absolute;
  top: 80px;
  right: -10px;
  padding: 10px 20px;
  background: #1e9bff;
  width: 200px;
  box-sizing: 0 5px 25px rgba(0, 0, 0, 0.1);
  border-radius: 15px;
  transition: 0.5s;
  visibility: hidden;
  opacity: 0;
}
.action .radio-buttons.menuActive {
  visibility: visible;
  opacity: 1;
}
.action .radio-buttons::before {
  content: "";
  position: absolute;
  top: -5px;
  right: 28px;
  width: 20px;
  height: 20px;
  background: #1e9bff;
  transform: rotate(45deg);
}
/*  */
.radio-buttons {
  flex: 3;
  flex-direction: column;
}
.radio-buttons p {
  color: #fff;
  font-size: 2em;
  margin-bottom: 10px;
  margin-top: 10px;
}
.radio-buttons label {
  cursor: pointer;
  font-size: 1.5em;
  color: #fff;
}
.radio-buttons label input {
  appearance: none;
  display: none;
}
.radio-buttons label span {
  position: relative;
  display: inline-block;
  width: 30px;
  height: 30px;
  margin-right: 15px;
  transition: 0.5s;
}
.radio-buttons label span::before {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: #fff;
  box-shadow: 0 -27px 0 #fff;
  transition: 0.5s;
}
.radio-buttons label input:checked ~ span.check::before {
  background: #0f0;
  box-shadow: 0 0 0 transparent;
}
.radio-buttons label span::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 3px;
  height: 100%;
  background: #fff;
  box-shadow: 27px 0 0 #fff;
  transition: 0.5s;
}
.radio-buttons label input:checked ~ span.check::after {
  height: 50%;
  background: #0f0;
  box-shadow: 0 0 0 transparent;
}
.radio-buttons label input:checked ~ span.check {
  transform: rotate(-45deg) translate(7px, -7px);
}

.time {
  bottom: 0px;
}
.videoList {
  text-align: left;
  list-style: none;
  margin: 10px;
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

.controls {
  width: 400px;
  border-radius: 10px;
  position: relative;
  bottom: -10vw;
  left: 50%;
  background-color: black;
  box-shadow: 3px 3px 5px black;
  transition: 1s all;
  display: flex;
  z-index: 10;
  transform: translateX(-50%);
}

.controls button {
  background: linear-gradient(to bottom, #222, #666);
}

.controls :before {
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

.controls button {
  height: 38px;
  line-height: 19px;
  box-shadow: inset 0 -5px 25px rgba(0, 0, 0, 0.3);
  border-right: 1px solid #333;
}
.timer {
  height: 38px;
  line-height: 19px;
  background: #8a8787;
  box-shadow: inset 0 -5px 25px #fff;
  border-right: 1px solid #333;
}

.controls button {
  position: relative;
  border: 0;
  flex: 1;
}

.play {
  border-radius: 10px 0 0 10px;
}

.fwd {
  border-radius: 0 10px 10px 0;
}

.timer {
  line-height: 100px;
  color: #fff;
  height: 5vw;
  position: relative;
}

.timer div {
  position: absolute;
  background-color: rgba(101, 98, 98, 0.362);
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

.controls > button:hover,
.controls > button:focus {
  box-shadow: inset 1px 1px 2px black;
}

.controls > button:active {
  box-shadow: inset 3px 3px 2px black;
}
</style>
