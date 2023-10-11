<template>
  <div class="player">
    <h1>VideoPlayer</h1>
    <video ref="media">
      <source src="../assets/sintel-short.mp4" type="video/mp4" />
      <source src="../assets/sintel-short.webm" type="video/webm" />
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
        @click="mediaBackward"
        data-icon="B"
        aria-label="rewind"
      ></button>
      <button
        class="fwd"
        @click="mediaForward"
        data-icon="F"
        aria-label="fast forward"
      ></button>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted } from "vue";

export default {
  setup() {
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

    const mediaBackward = () => {
      clearIntervalRwdFwd();

      if (controls.value && controls.value.classList.contains("active")) {
        controls.value.classList.remove("active");
        if (media.value) {
          media.value.play();
        }
      } else {
        if (controls.value) {
          controls.value.classList.add("active");
        }
        if (media.value) {
          media.value.pause();
          intervalRwd.value = setInterval(windBackward, 200);
        }
      }
    };

    const mediaForward = () => {
      clearIntervalRwdFwd();

      if (controls.value && controls.value.classList.contains("active")) {
        controls.value.classList.remove("active");
        if (media.value) {
          media.value.play();
        }
      } else {
        if (controls.value) {
          controls.value.classList.add("active");
        }
        if (media.value) {
          media.value.pause();
          intervalFwd.value = setInterval(windForward, 200);
        }
      }
    };

    const windBackward = () => {
      if (media.value) {
        if (media.value.currentTime <= 3) {
          stopMedia();
        } else {
          media.value.currentTime -= 3;
        }
      }
    };

    const windForward = () => {
      if (media.value) {
        if (media.value.currentTime >= media.value.duration - 3) {
          stopMedia();
        } else {
          media.value.currentTime += 3;
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

    onMounted(() => {
      if (media.value) {
        media.value.addEventListener("timeupdate", setTime);
      }
    });

    return {
      media,
      controls,
      timerWrapper,
      timerBar,
      formattedTime,
      togglePlay,
      stopMedia,
      mediaBackward,
      mediaForward,
      seekToTime,
      showTimeOnHover,
      hideTimeOnLeave,
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
}

p {
  position: absolute;
  top: 310px;
}

.player {
  position: absolute;
}

.controls {
  opacity: 0;
  width: 400px;
  border-radius: 10px;
  position: absolute;
  bottom: 20px;
  left: 50%;
  margin-left: -200px;
  background-color: black;
  box-shadow: 3px 3px 5px black;
  transition: 1s all;
  display: flex;
}

button,
.controls {
  background: linear-gradient(to bottom, #222, #666);
}

.player:hover .controls,
player:focus .controls {
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

/* .active:before {
  color: red;
} */
</style>
