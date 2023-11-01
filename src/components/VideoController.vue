<template>
  <div class="controls" ref="controlEl" @keydown="handleKeydown">
    <button class="play" @click="togglePlay" data-icon="P"></button>
    <button class="stop" @click="toggleStop" data-icon="S"></button>
    <button class="rwd" @click="goBackward" data-icon="B"></button>
    <button class="fwd" @click="goForward" data-icon="F"></button>
    <button class="rrwd" @click="goFpsBackward" data-icon="p"></button>
    <button class="ffwd" @click="goFpsForward" data-icon="j"></button>
    <button class="volume" @click="toggleVolume" data-icon="g"></button>
    <input
      v-if="state.showVolume"
      type="range"
      class="volumeBar"
      min="0"
      max="1"
      step="0.01"
      v-model="state.volume"
      @input="setVolume"
    />
  </div>
  <div class="volume-level" v-if="state.showVolume">{{ volumeLevel }}</div>
</template>

<script lang="ts">
import {
  defineComponent,
  PropType,
  reactive,
  watch,
  onUnmounted,
  computed,
} from "vue";
interface State {
  interval: number;
  volume: number;
  showVolume: boolean;
}
interface Video {
  src: string;
  name: string;
  fps: number;
  currentTime: number;
}
export default defineComponent({
  props: {
    videoEl: {
      type: Object as PropType<HTMLVideoElement | null>,
      required: true,
    },
    autoPlay: {
      type: Boolean,
      required: true,
    },
    videoData: {
      type: Object as () => { videoIndex: number; IVideo: Video[] },
      required: true,
    },
  },
  emits: [
    "resetVideo",
    "backwardVideo",
    "forwardVideo",
    "endVideo",
    "firstStop",
    "fpsBackwardVideo",
    "fpsForwardVideo",
    "setVolume",
  ],
  setup(props, { emit }) {
    const state = reactive<State>({
      interval: 10,
      volume: 1,
      showVolume: false,
    });
    const toggleVolume = () => {
      state.showVolume = !state.showVolume;
    };
    const setVolume = (event: Event) => {
      const target = event.target as HTMLInputElement;
      if (target) {
        state.volume = parseFloat(target.value);
        if (props.videoEl) {
          emit("setVolume", state.volume);
        }
      }
    };
    const volumeLevel = computed(() => `${Math.round(state.volume * 100)}%`);

    const handleKeydown = (event: KeyboardEvent) => {
      switch (event.key) {
        case "ArrowLeft":
          if (event.ctrlKey) goFpsBackward();
          else goBackward();
          break;
        case "ArrowRight":
          if (event.ctrlKey) goFpsForward();
          else goForward();
          break;
        case " ":
          togglePlay();
          break;
        case "Enter":
          toggleStop();
          break;
      }
    };
    document.addEventListener("keydown", handleKeydown);
    onUnmounted(() => {
      document.removeEventListener("keydown", handleKeydown);
    });
    const togglePlay = () => {
      if (props.videoEl) {
        if (props.videoEl.paused) {
          props.videoEl.play();
        } else {
          props.videoEl.pause();
        }
      }
    };

    const toggleStop = () => {
      if (props.videoEl) {
        props.videoEl.pause();
        emit("resetVideo");
      }
    };

    const firstStop = () => {
      if (props.videoEl) {
        props.videoEl.pause();
        emit("firstStop");
      }
    };

    const goBackward = () => {
      if (props.videoEl) {
        if (props.videoEl.currentTime <= state.interval) {
          emit("resetVideo");
          toggleStop();
        } else {
          emit("backwardVideo", state.interval);
        }
      }
    };

    const goForward = () => {
      if (props.videoEl) {
        if (
          props.videoEl.currentTime >=
          props.videoEl.duration - state.interval
        ) {
          emit("endVideo");
        } else {
          emit("forwardVideo", state.interval);
        }
      }
    };

    const goFpsBackward = () => {
      if (props.videoEl) {
        const videoFps = props.videoData.IVideo[props.videoData.videoIndex].fps;
        const frameDuration = 1 / videoFps;
        if (props.videoEl.currentTime <= frameDuration) {
          emit("resetVideo");
          toggleStop();
        } else {
          emit("fpsBackwardVideo", frameDuration);
        }
      }
    };

    const goFpsForward = () => {
      if (props.videoEl) {
        const videoFps = props.videoData.IVideo[props.videoData.videoIndex].fps;
        const frameDuration = 1 / videoFps;
        if (
          props.videoEl.currentTime >=
          props.videoEl.duration - frameDuration
        ) {
          emit("endVideo");
        } else {
          emit("fpsForwardVideo", frameDuration);
        }
      }
    };

    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl) {
          newVideoEl.volume = state.volume;
          newVideoEl.addEventListener("loadedmetadata", () => {
            if (props.autoPlay) {
              togglePlay();
            } else {
              firstStop();
            }
          });
        }
      },
      { immediate: true }
    );
    return {
      state,
      goForward,
      goBackward,
      togglePlay,
      toggleStop,
      firstStop,
      goFpsBackward,
      goFpsForward,
      handleKeydown,
      toggleVolume,
      setVolume,
      volumeLevel,
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
.controls button {
  position: relative;
  border: 0;
  flex: 1;
}
.play {
  border-radius: 10px 0 0 10px;
}
.volume {
  border-radius: 0 10px 10px 0;
}
.volumeBar {
  position: absolute;
  left: 26vw;
  bottom: 0.5vw;
  cursor: pointer;
}
.volume-level {
  color: #fff;
  position: absolute;
  display: inline-block;
  left: 40vw;
  bottom: 4vw;
}
.controls > button:hover,
.controls > button:focus {
  box-shadow: inset 1px 1px 2px black;
}
</style>
