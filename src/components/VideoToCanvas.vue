<template>
  <div class="test">
    <video ref="videoEl">
      <source type="video/mp4" />
    </video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, PropType, reactive, watch } from "vue";
interface State {
  animationFrameId: number;
}
export default defineComponent({
  props: {
    videoEl: {
      type: Object as PropType<HTMLVideoElement | null>,
      /* [Object, null] */
      required: true,
    },
  },
  setup(props) {
    const state = reactive<State>({
      animationFrameId: 0,
    });
    const localVideoEl = ref<HTMLVideoElement | null>(props.videoEl);
    const canvas = ref<HTMLCanvasElement | null>(null);
    const ctx = ref<CanvasRenderingContext2D | null>(null);
    const drawCanvas = () => {
      if (canvas.value && ctx.value && localVideoEl.value) {
        const videoAspectRatio =
          localVideoEl.value.videoWidth / localVideoEl.value.videoHeight;
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
          localVideoEl.value,
          xStart,
          yStart,
          drawWidth,
          drawHeight
        );
        if (localVideoEl.value.paused) {
          cancelAnimationFrame(state.animationFrameId);
        } else {
          state.animationFrameId = requestAnimationFrame(drawCanvas);
        }
      }
    };

    watch(
      () => localVideoEl.value,
      (newValue) => {
        console.log("adf");
        if (newValue) {
          localVideoEl.value = newValue;
        }
        if (canvas.value) {
          ctx.value = canvas.value.getContext("2d") as CanvasRenderingContext2D;
        }
        localVideoEl.value?.addEventListener("loadedmetadata", () => {
          if (canvas.value && localVideoEl.value && ctx.value) {
            const computedStyle = getComputedStyle(localVideoEl.value);
            canvas.value.width = parseInt(computedStyle.width, 10);
            canvas.value.height = parseInt(computedStyle.height, 10);
            localVideoEl.value.addEventListener("timeupdate", () => {
              drawCanvas();
            });
          }
        });
      }
    );

    return { ctx, canvas };
  },
});
</script>

<style scoped>
canvas {
  height: 50vh;
  width: 100vh;
}
video {
  height: 50vh;
  width: 100vh;
  /* display: none; */
}
</style>
