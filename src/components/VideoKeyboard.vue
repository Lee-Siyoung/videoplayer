<template>
  <div></div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive } from "vue";
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
    const state = reactive<State>({
      animationFrameId: 0,
    });

    return {};
  },
});
</script>

<style scoped></style>
