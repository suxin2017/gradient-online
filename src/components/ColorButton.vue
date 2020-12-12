<template>
  <div
    class="select-panel-banner-button"
    :class="{ active: active }"
    :style="{ background: color, left: `${left}px`, zIndex: zIndex }"
    ref="button"
    tabindex="0"
    @mousedown="mousedown"
  ></div>
</template>

<script>
import { onMounted, onUnmounted, ref } from "vue";
import { SELECT_WIDTH } from "../App.vue";
export default {
  props: {
    color: String,
    initLeft: Number,
    zIndex: Number,
    active: Boolean,
  },
  setup(props, context) {
    const isMouseDown = ref(false);
    const button = ref(null);
    const left = ref(props.initLeft);
    const baseOffset = ref(0);
    const currentPosition = ref(0);
    console.log(left.value);
    function mousedown(e) {
      baseOffset.value = e.clientX;
      currentPosition.value = left.value;
      isMouseDown.value = true;
    }
    function unsetMouseDown() {
      isMouseDown.value = false;
    }
    const mouseMove = (e) => {
      if (isMouseDown.value === true) {
        const targetOffset =
          currentPosition.value + e.clientX - baseOffset.value;
        console.log("鼠标点击中", targetOffset);

        if (targetOffset < 0 || targetOffset > SELECT_WIDTH) {
          return;
        }

        left.value = targetOffset;
        context.emit("onMove", targetOffset);
      }
      //   console.log(e, isMouseDown);
    };
    onMounted(() => {
      window.addEventListener("mouseup", unsetMouseDown);
      window.addEventListener("mousemove", mouseMove);
    });
    onUnmounted(() => {
      window.removeEventListener("mouseup", unsetMouseDown);
      window.removeEventListener("mousemove", unsetMouseDown);
    });

    return { isMouseDown, button, left, mousedown };
  },
};
</script>

<style>
.select-panel-banner-button {
  height: 22px;
  width: 10px;
  border-radius: 2px;
  position: absolute;
  top: -2px;
  cursor: pointer;
}
.select-panel-banner-button:focus,
.active {
  outline: 1px solid #fff;
  box-shadow: 0 0 4px 0px black;
}
</style>