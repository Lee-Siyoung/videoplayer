<template>
  <button @click="capture">Capture</button>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";

export default defineComponent({
  props: {
    canvas: {
      type: Object as PropType<HTMLCanvasElement | null>,
      required: true,
    },
    mediaEl: {
      type: Object as PropType<HTMLVideoElement | null>,
      required: true,
    },
    ctx: {
      type: Object as PropType<CanvasRenderingContext2D | null>,
      required: true,
    },
  },
  setup(props) {
    const capture = () => {
      const { ctx, mediaEl, canvas } = props;
      if (ctx && mediaEl && canvas) {
        const tempCanvas = document.createElement("canvas");
        const tempCtx = tempCanvas.getContext("2d");
        tempCanvas.width = canvas.width;
        tempCanvas.height = canvas.height;
        tempCtx?.drawImage(canvas, 0, 0);

        canvas.width = mediaEl.videoWidth;
        canvas.height = mediaEl.videoHeight;
        ctx.drawImage(mediaEl, 0, 0, canvas.width, canvas.height);
        ctx.drawImage(tempCanvas, 0, 0, canvas.width, canvas.height);
        canvas.toBlob((blob) => {
          if (blob && canvas && mediaEl && ctx) {
            const imageUrl = URL.createObjectURL(blob);

            const link = document.createElement("a");
            link.href = imageUrl;
            link.download = "captured_frame.png";
            link.click();

            URL.revokeObjectURL(imageUrl);

            canvas.width = tempCanvas.width;
            canvas.height = tempCanvas.height;
            ctx.drawImage(tempCanvas, 0, 0);
          }
        }, "image/png");
      }
    };

    return { capture };
  },
});
</script>
