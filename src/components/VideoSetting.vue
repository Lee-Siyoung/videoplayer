<template>
  <div class="action">
    <div class="profile" @click="menuToggle">
      <img src="../assets/setting.png" />
    </div>
    <div class="radio-buttons" :class="{ menuActive: state.isActive }">
      <p>시작 설정</p>
      <label>
        <input
          type="radio"
          id="play"
          name="videoControl"
          value="true"
          @change="state.autoPlay = true"
        />
        <span class="check"></span>시작
      </label>
      <label>
        <input
          type="radio"
          id="stop"
          name="videoControl"
          value="false"
          @change="state.autoPlay = false"
          checked
        />
        <span class="check"></span>멈춤</label
      >
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, reactive, watch } from "vue";
interface State {
  autoPlay: boolean;
  isActive: boolean;
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
      autoPlay: false,
      isActive: false,
    });
    const menuToggle = () => {
      state.isActive = !state.isActive;
    };
    watch(
      () => props.videoEl,
      (newVideoEl) => {
        if (newVideoEl) {
          newVideoEl.addEventListener("loadedmetadata", () => {
            if (state.autoPlay) {
              /* togglePlay(); */
            } else {
              /* toggleStop(); */
            }
          });
        }
      },
      { immediate: true }
    );
    return { state, menuToggle };
  },
});
</script>

<style scoped>
.action {
  position: relative;
  top: 20px;
  left: 400px;
  height: 30vw;
}
.action .profile {
  position: relative;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  overflow: hidden;
  cursor: pointer;
}
.action .profile img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: invert(99%) sepia(0%) saturate(3993%) hue-rotate(209deg)
    brightness(117%) contrast(100%);
}
.action .radio-buttons {
  position: absolute;
  top: 80px;
  right: -10px;
  padding: 10px 20px;
  background: #1e9bff;
  width: 200px;
  box-sizing: 0 5px 25px rgba(0, 0, 0, 0.1);
  border-radius: 15px;
  transition: 0.5s;
  visibility: hidden;
  opacity: 0;
}
.action .radio-buttons.menuActive {
  visibility: visible;
  opacity: 1;
}
.action .radio-buttons::before {
  content: "";
  position: absolute;
  top: -5px;
  right: 28px;
  width: 20px;
  height: 20px;
  background: #1e9bff;
  transform: rotate(45deg);
}
.radio-buttons {
  flex: 3;
  flex-direction: column;
}
.radio-buttons p {
  color: #fff;
  font-size: 2em;
  margin-bottom: 10px;
  margin-top: 10px;
  letter-spacing: 0.1em;
  font-weight: 400;
}
.radio-buttons label {
  cursor: pointer;
  font-size: 1.5em;
  color: #fff;
}
.radio-buttons label input {
  appearance: none;
  display: none;
}
.radio-buttons label span {
  position: relative;
  display: inline-block;
  width: 30px;
  height: 30px;
  margin-right: 15px;
  transition: 0.5s;
}
.radio-buttons label span::before {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: #fff;
  box-shadow: 0 -27px 0 #fff;
  transition: 0.5s;
}
.radio-buttons label input:checked ~ span.check::before {
  background: #0f0;
  box-shadow: 0 0 0 transparent;
}
.radio-buttons label span::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 3px;
  height: 100%;
  background: #fff;
  box-shadow: 27px 0 0 #fff;
  transition: 0.5s;
}
.radio-buttons label input:checked ~ span.check::after {
  height: 50%;
  background: #0f0;
  box-shadow: 0 0 0 transparent;
}
.radio-buttons label input:checked ~ span.check {
  transform: rotate(-45deg) translate(7px, -7px);
}
</style>
