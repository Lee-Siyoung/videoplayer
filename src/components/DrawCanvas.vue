<template>
  <div></div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive, watch, ref } from "vue";
interface State {
  animationFrameId: number;
}
export default defineComponent({
  props: {
    videoEl: {
      type: Object as PropType<HTMLVideoElement | null>,
      required: true,
    },
    canvas: {
      type: Object as PropType<HTMLCanvasElement | null>,
      required: true,
    },
    ctx: {
      type: Object as PropType<CanvasRenderingContext2D | null>,
      required: true,
    },
  },
  setup(props) {
    console.log("drawcanvas");
    const localCtx = ref<CanvasRenderingContext2D | null>(props.ctx);
    const localCanvas = ref<HTMLCanvasElement | null>(props.canvas);
    const state = reactive<State>({
      animationFrameId: 0,
    });
    const drawCanvas = () => {
      if (localCanvas.value && localCtx.value && props.videoEl) {
        const videoAspectRatio =
          props.videoEl.videoWidth / props.videoEl.videoHeight;
        const canvasAspectRatio =
          localCanvas.value.width / localCanvas.value.height;
        const { drawWidth, drawHeight, xStart, yStart } = (() => {
          if (videoAspectRatio < canvasAspectRatio) {
            const height = localCanvas.value.height;
            const width = height * videoAspectRatio;
            return {
              drawWidth: width,
              drawHeight: height,
              xStart: (localCanvas.value.width - width) / 2,
              yStart: 0,
            };
          } else {
            const width = localCanvas.value.width;
            const height = width / videoAspectRatio;
            return {
              drawWidth: width,
              drawHeight: height,
              xStart: 0,
              yStart: (localCanvas.value.height - height) / 2,
            };
          }
        })();
        localCtx.value.drawImage(
          props.videoEl,
          xStart,
          yStart,
          drawWidth,
          drawHeight
        );
        if (props.videoEl.paused) {
          cancelAnimationFrame(state.animationFrameId);
        } else {
          state.animationFrameId = requestAnimationFrame(drawCanvas);
        }
      }
    };

    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl && localCanvas.value) {
          localCtx.value = localCanvas.value.getContext("2d");
          newVideoEl.addEventListener("loadedmetadata", () => {
            if (localCanvas.value && newVideoEl && localCtx.value) {
              const computedStyle = getComputedStyle(newVideoEl);
              localCanvas.value.width = parseInt(computedStyle.width, 10);
              localCanvas.value.height = parseInt(computedStyle.height, 10);
              newVideoEl.addEventListener("timeupdate", () => {
                drawCanvas();
              });
            }
          });
        }
      },
      { immediate: true }
    );

    return { drawCanvas, localCtx, localCanvas };
  },
});
</script>

<style scoped></style>
