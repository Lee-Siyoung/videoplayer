<template>
  <div class="timer" ref="timerWrapper" @click="seekToTime">
    <div ref="progressbar"></div>
    <span>{{ state.timeCode }}</span>
    <span class="time"
      >{{ state.formattedTime }} / {{ state.videoDuration }}</span
    >
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive, ref, watch } from "vue";
interface State {
  videoDuration: string;
  formattedTime: string;
  timeCode: string;
  isDragging: boolean;
}
export default defineComponent({
  props: {
    videoEl: {
      type: Object as PropType<HTMLVideoElement | null>,
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
    const timerWrapper = ref<HTMLElement | null>(null);
    const progressbar = ref<HTMLElement | null>(null);
    const setTime = () => {
      if (props.videoEl && timerWrapper.value && progressbar.value) {
        /* state.timeCode = smpteTimeCode(props.videoEl.currentTime); */
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

        const barLength =
          (props.videoEl.currentTime / props.videoEl.duration) *
            timerWrapper.value.clientWidth || 0;
        progressbar.value.style.width = barLength + "px";
      }
    };

    const seekToTime = (event: MouseEvent) => {
      if (props.videoEl && timerWrapper.value) {
        const barRect = timerWrapper.value.getBoundingClientRect();
        const clickX = event.clientX - barRect.left;
        const barWidth = barRect.width;
        const newTime = (clickX / barWidth) * props.videoEl.duration;
        emit("seekVideo", newTime);
        //drawCanvas();
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
    /* const smpteTimeCode = (currentTime: number) => {
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
    */
    const update = () => {
      if (props.videoEl) {
        /* state.timeCode = smpteTimeCode(props.videoEl.currentTime); */
      }
    };

    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl) {
          timerWrapper.value?.addEventListener("mousedown", startDrag);
          timerWrapper.value?.addEventListener("mousemove", duringDrag);
          timerWrapper.value?.addEventListener("mouseup", endDrag);
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
    return { state, seekToTime };
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
