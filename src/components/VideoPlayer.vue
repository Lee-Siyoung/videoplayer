<template>
  <div class="player">
    <div class="video-wrapper">
      <h1>Movie</h1>
      <button @click="changeMedia" data-icon="C" aria-label="change media">
        Change Media
      </button>
      <video ref="media">
        <source src="" type="video/mp4" />
      </video>
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
          <div ref="timerBar"></div>
          <span aria-label="timer">{{ formattedTime }}</span>
        </div>
        <button
          class="rwd"
          @click="windBackward"
          data-icon="B"
          aria-label="rewind"
        ></button>
        <button
          class="fwd"
          @click="windForward"
          data-icon="F"
          aria-label="fast forward"
        ></button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted } from "vue";

export default {
  setup() {
    const fileNames = ref<string[]>([]);
    const mediaIndex = ref(0);
    const media = ref<HTMLVideoElement | null>(null);
    const controls = ref<HTMLElement | null>(null);
    const timerWrapper = ref<HTMLElement | null>(null);
    const timerBar = ref<HTMLElement | null>(null);
    const isPlaying = ref(false);
    const formattedTime = ref("00:00");
    const intervalFwd = ref<number | null>(null);
    const intervalRwd = ref<number | null>(null);

    const togglePlay = () => {
      if (media.value) {
        clearIntervalRwdFwd();
        if (media.value.paused) {
          media.value.play();
          isPlaying.value = true;
        } else {
          media.value.pause();
          isPlaying.value = false;
        }
      }
    };

    const stopMedia = () => {
      if (media.value) {
        clearIntervalRwdFwd();
        media.value.pause();
        media.value.currentTime = 0;
        isPlaying.value = false;
      }
    };

    const clearIntervalRwdFwd = () => {
      if (intervalRwd.value) {
        clearInterval(intervalRwd.value);
      }
      if (intervalFwd.value) {
        clearInterval(intervalFwd.value);
      }
    };

    const windBackward = () => {
      if (media.value) {
        if (media.value.currentTime <= 10) {
          stopMedia();
        } else {
          media.value.currentTime -= 10;
        }
      }
    };

    const windForward = () => {
      if (media.value) {
        if (media.value.currentTime >= media.value.duration - 10) {
          stopMedia();
        } else {
          media.value.currentTime += 10;
        }
      }
    };

    const setTime = () => {
      if (media.value && timerWrapper.value && timerBar.value) {
        const minutes = Math.floor(media.value.currentTime / 60);
        const seconds = Math.floor(media.value.currentTime - minutes * 60);

        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");

        const mediaTime = `${minuteValue}:${secondValue}`;
        formattedTime.value = mediaTime;

        const barLength =
          (media.value.currentTime / media.value.duration) *
            timerWrapper.value.clientWidth || 0;
        timerBar.value.style.width = barLength + "px";
      }
    };

    const seekToTime = (event: MouseEvent) => {
      if (media.value && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const clickX = event.clientX - barRect.left;
        const barWidth = barRect.width;
        const newTime = (clickX / barWidth) * media.value.duration;
        media.value.currentTime = newTime;
      }
    };

    const showTimeOnHover = (event: MouseEvent) => {
      if (media.value && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const hoverX = event.clientX - barRect.left;
        const barWidth = barRect.width;

        const hoverTime = (hoverX / barWidth) * media.value.duration;
        const minutes = Math.floor(hoverTime / 60);
        const seconds = Math.floor(hoverTime - minutes * 60);
        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");
        formattedTime.value = `${minuteValue}:${secondValue}`;
      }
    };

    const hideTimeOnLeave = () => {
      if (media.value) {
        setTime();
      }
    };

    const changeMedia = () => {
      mediaIndex.value = (mediaIndex.value + 1) % fileNames.value.length;

      if (media.value) {
        media.value.src = fileNames.value[mediaIndex.value];
        media.value.load();
        media.value.play();
      }
      console.log(mediaIndex.value);
    };

    onMounted(() => {
      const context = require.context("../assets", false, /\.(mp4|webm)$/);
      const filenames = context
        .keys()
        .map((key) => key.replace("./", "../assets/"));
      fileNames.value = filenames;
      if (media.value && fileNames.value.length > 0) {
        media.value.src = fileNames.value[mediaIndex.value];
        media.value.load();
      }

      if (media.value) {
        media.value.addEventListener("timeupdate", setTime);
      }
    });

    return {
      fileNames,
      media,
      controls,
      timerWrapper,
      timerBar,
      formattedTime,
      windForward,
      windBackward,
      togglePlay,
      stopMedia,
      seekToTime,
      showTimeOnHover,
      hideTimeOnLeave,
      changeMedia,
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

video {
  border: 1px solid black;
  width: 100vh;
  height: 80vh;
  display: block;
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
