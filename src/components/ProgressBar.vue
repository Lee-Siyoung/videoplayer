<template>
  <div
    class="timer"
    @mousedown="startDrag"
    @mousemove="duringDrag"
    @mouseup="endDrag"
    @click="seekToTime"
  >
    <div class="progressbar"></div>
    <span>{{ state.timeCode }}</span>
    <span class="time"
      >{{ state.formattedTime }} / {{ state.videoDuration }}</span
    >
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive, watch, onUpdated } from "vue";
interface State {
  videoDuration: string;
  formattedTime: string;
  timeCode: string;
  isDragging: boolean;
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
    videoData: {
      type: Object as () => { videoIndex: number; IVideo: Video[] },
      required: true,
    },
  },
  emits: ["seekVideo"],
  setup(props, { emit }) {
    const state = reactive<State>({
      videoDuration: "00:00",
      formattedTime: "00:00",
      timeCode: "00:00:00:00",
      isDragging: false,
    });

    const setTime = () => {
      if (props.videoEl) {
        state.timeCode = smpteTimeCode(props.videoEl.currentTime);
        const minutes = Math.floor(props.videoEl.currentTime / 60);
        const seconds = Math.floor(props.videoEl.currentTime - minutes * 60);
        const totalMinutes = Math.floor(props.videoEl.duration / 60);
        const totalSeconds = Math.floor(props.videoEl.duration % 60);

        const minuteValue = minutes.toString().padStart(2, "0");
        const secondValue = seconds.toString().padStart(2, "0");
        const totalMinuteValue = totalMinutes.toString().padStart(2, "0");
        const totalSecondValue = totalSeconds.toString().padStart(2, "0");

        const videoTime = `${minuteValue}:${secondValue}`;
        state.formattedTime = videoTime;

        const videoDuration = `${totalMinuteValue}:${totalSecondValue}`;
        state.videoDuration = videoDuration;
      }
    };

    const seekToTime = (event: MouseEvent) => {
      if (props.videoEl) {
        const targetElement = event.currentTarget as HTMLElement;
        const barRect = targetElement.getBoundingClientRect();
        const clickX = event.clientX - barRect.left;
        const barWidth = barRect.width;
        const newTime = (clickX / barWidth) * props.videoEl.duration;
        emit("seekVideo", newTime);
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
    const smpteTimeCode = (currentTime: number) => {
      const videoFps = props.videoData.IVideo[props.videoData.videoIndex].fps;
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
      if (props.videoEl) {
        state.timeCode = smpteTimeCode(props.videoEl.currentTime);
      }
    };
    onUpdated(() => {
      const timerWrapper = document.querySelector(".timer") as HTMLElement;
      const progressbar = document.querySelector(".progressbar") as HTMLElement;
      if (timerWrapper && progressbar && props.videoEl) {
        const barLength =
          (props.videoEl.currentTime / props.videoEl.duration) *
            timerWrapper.clientWidth || 0;
        progressbar.style.width = barLength + "px";
      }
    });
    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl) {
          newVideoEl.addEventListener("loadedmetadata", () => {
            setTime();
            setInterval(update, 1000 / 24);
            newVideoEl.addEventListener("timeupdate", () => {
              setTime();
            });
          });
        }
      },
      { immediate: true }
    );
    return { state, seekToTime, startDrag, duringDrag, endDrag };
  },
});
</script>

<style scoped>
.timer {
  height: 38px;
  line-height: 19px;
  background: #8a8787;
  box-shadow: inset 0 -5px 25px #fff;
  border-right: 1px solid #333;
}
.timer {
  line-height: 100px;
  color: #fff;
  height: 5vw;
  position: relative;
}

.progressbar {
  background: #fff;
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
.time {
  bottom: 0px;
}
</style>
