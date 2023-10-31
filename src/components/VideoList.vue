<template>
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
</template>

<script lang="ts">
import { defineComponent, reactive, onMounted, PropType, watch } from "vue";

interface IVideo {
  src: string;
  name: string;
  fps: number;
  currentTime: number;
}

interface State {
  videoIndex: number;
  IVideo: IVideo[];
}

export default defineComponent({
  props: {
    videoEl: {
      type: Object as PropType<HTMLVideoElement | null>,
      required: true,
    },
  },
  emits: ["updateSrc", "updateState", "currentTime"],
  setup(props, { emit }) {
    const state = reactive<State>({
      videoIndex: 0,
      IVideo: [
        {
          src: "",
          name: "",
          fps: 24,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 25,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 23.98,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 29.97,
          currentTime: 0,
        },
        {
          src: "",
          name: "",
          fps: 24,
          currentTime: 0,
        },
      ],
    });
    let a = 0;
    const clickVideo = (index: number) => {
      state.videoIndex = index;
      const newSrc = state.IVideo[state.videoIndex].src;
      emit("currentTime", props.videoEl?.currentTime, a);

      emit("updateSrc", newSrc);
    };
    watch(state, (newState) => {
      a = newState.videoIndex;
      emit("updateState", state);
    });
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
      const initialSrc = state.IVideo[state.videoIndex].src;
      emit("updateSrc", initialSrc);
    });
    return { state, clickVideo };
  },
});
</script>
<style scoped>
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
  cursor: pointer;
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
.scroll::-webkit-scrollbar {
  width: 0.5vw;
}
.scroll::-webkit-scrollbar-thumb {
  background-color: #1e9bff;
}
.scroll::-webkit-scrollbar-track {
  background-color: #fff;
}
</style>
