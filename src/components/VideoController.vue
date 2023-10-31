<template>
  <div class="controls" ref="controlEl">
    <button class="play" @click="togglePlay" data-icon="P"></button>
    <button class="stop" @click="toggleStop" data-icon="S"></button>
    <button class="rwd" @click="goBackward" data-icon="B"></button>
    <button class="fwd" @click="goForward" data-icon="F"></button>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive, watch } from "vue";
interface State {
  interval: number;
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
  },
  emits: ["resetVideo", "backwardVideo", "forwardVideo", "endVideo"],
  setup(props, { emit }) {
    const state = reactive<State>({
      interval: 10,
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

    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl) {
          newVideoEl.addEventListener("loadedmetadata", () => {
            if (props.autoPlay) {
              togglePlay();
            } else {
              toggleStop();
            }
          });
        }
      },
      { immediate: true }
    );
    return { goForward, goBackward, togglePlay, toggleStop };
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
.fwd {
  border-radius: 0 10px 10px 0;
}
.controls > button:hover,
.controls > button:focus {
  box-shadow: inset 1px 1px 2px black;
}
</style>
