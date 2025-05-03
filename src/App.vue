<script setup>
import { ref, computed } from "vue";

const RADIUS = 80;

const circle1 = ref({ x: 50, y: 50 });
const circle2 = ref({ x: 200, y: 50 });
const dragging = ref(null);

function startDrag(event, id) {
  const circle = id === 1 ? circle1.value : circle2.value;
  const offsetX = event.clientX - circle.x;
  const offsetY = event.clientY - circle.y;

  dragging.value = { id, offsetX, offsetY };
}

function onMouseMove(event) {
  if (!dragging.value) return;

  const { id, offsetX, offsetY } = dragging.value;
  const newX = event.clientX - offsetX;
  const newY = event.clientY - offsetY;

  if (id === 1) {
    circle1.value = { x: newX, y: newY };
  } else {
    circle2.value = { x: newX, y: newY };
  }
}

function stopDrag() {
  dragging.value = null;
}

const isTouching = computed(() => {
  const r = RADIUS / 2;

  const center_x_1 = circle1.value.x + r;
  const center_y_1 = circle1.value.y + r;
  const center_x_2 = circle2.value.x + r;
  const center_y_2 = circle2.value.y + r;

  const dx = center_x_1 - center_x_2;
  const dy = center_y_1 - center_y_2;

  const distance = Math.sqrt(dx * dx + dy * dy);
  return distance < RADIUS;
});

const circle1ZIndex = computed(() =>
  dragging.value && dragging.value.id === 1 ? 10 : 1
);
const circle2ZIndex = computed(() =>
  dragging.value && dragging.value.id === 2 ? 10 : 1
);
</script>

<template>
  <div id="app" @mousemove="onMouseMove" @mouseup="stopDrag">
    <div class="indicator">
      Circles are {{ isTouching ? "touching" : "not touching" }}
    </div>
    <div
      class="circle"
      :style="{
        left: circle1.x + 'px',
        top: circle1.y + 'px',
        zIndex: circle1ZIndex
      }"
      @mousedown="startDrag($event, 1)"
    ></div>

    <div
      class="circle circle2"
      :style="{
        left: circle2.x + 'px',
        top: circle2.y + 'px',
        zIndex: circle2ZIndex
      }"
      @mousedown="startDrag($event, 2)"
    ></div>
  </div>
</template>

<style scoped>
#app {
  width: 100vw;
  height: 100vh;
  background-color: #f0f0f0;
  position: relative;
  overflow: hidden;
  user-select: none;
}

.indicator {
  position: absolute;
  top: 10px;
  left: 10px;
  color: black;
  background: white;
  padding: 6px 12px;
  border-radius: 6px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  font-weight: bold;
  z-index: 10;
}

.circle {
  position: absolute;
  width: 80px;
  height: 80px;
  background-color: red;
  border-radius: 50%;
  cursor: grab;
}

.circle2 {
  background-color: green;
}
</style>
