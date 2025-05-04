<script setup>
import { ref, computed } from "vue";

const RADIUS = 80;
const dragging = ref(null);

const circle1 = ref({ x: 50, y: 150 });

const red_count = ref(2);
const red_circles = ref([
  { x: 200, y: 150 },
  { x: 200, y: 250 }
]);

function removeRedCircle() {
  red_count.value--;
  red_circles.value.pop();
}

function addRedCircle() {
  red_count.value++;
  let new_circle = JSON.parse(JSON.stringify(red_circles.value.at(-1)));
  new_circle.y += 100;
  red_circles.value.push(new_circle);
}

const disableAddButton = computed(() => {
  return !(red_count.value < 5);
});
const disableDeleteButton = computed(() => {
  return !(red_count.value > 1);
});

function startDrag(event, id) {
  const circle = id === -1 ? circle1.value : red_circles.value[id];
  const offsetX = event.clientX - circle.x;
  const offsetY = event.clientY - circle.y;

  dragging.value = { id, offsetX, offsetY };
}

function onMouseMove(event) {
  if (!dragging.value) return;

  const { id, offsetX, offsetY } = dragging.value;
  const newX = event.clientX - offsetX;
  const newY = event.clientY - offsetY;

  if (id === -1) {
    circle1.value = { x: newX, y: newY };
  } else {
    red_circles.value[id] = { x: newX, y: newY };
  }
}

function stopDrag() {
  dragging.value = null;
}

const isTouching = computed(() => {
  const r = RADIUS / 2;

  //green circle
  const center_x_1 = circle1.value.x + r;
  const center_y_1 = circle1.value.y + r;

  //red circles
  for (let c of red_circles.value) {
    const center_x_2 = c.x + r;
    const center_y_2 = c.y + r;

    const dx = center_x_1 - center_x_2;
    const dy = center_y_1 - center_y_2;
    const distance = Math.sqrt(dx * dx + dy * dy);
    if (distance < RADIUS) {
      return true;
    }
  }

  return false;
});

//Implementation of higher Z index for the grabbed circle
// const circle1ZIndex = computed(() =>
//   dragging.value && dragging.value.id === -1 ? 10 : 1
// );
// const circle2ZIndex = computed(() =>
//   dragging.value && dragging.value.id === 2 ? 10 : 1
// );
</script>

<template>
  <div id="app" @mousemove="onMouseMove" @mouseup="stopDrag">
    <div class="indicator">
      <div>
        Green is {{ isTouching ? "touching" : "not touching" }} a red circle
      </div>
      <button
        :disabled="disableAddButton"
        class="addButton"
        @click="addRedCircle()"
      >
        Add Circle
      </button>
      <button :disabled="disableDeleteButton" @click="removeRedCircle()">
        Remove Circle
      </button>
    </div>
    <div
      class="circle"
      :style="{
        left: circle1.x + 'px',
        top: circle1.y + 'px',
        zIndex: 1
      }"
      @mousedown="startDrag($event, -1)"
    ></div>

    <!-- Red circles loop to show all circles -->
    <div
      class="circle circle2"
      v-for="(circ, index) in red_circles"
      :style="{
        left: circ.x + 'px',
        top: circ.y + 'px',
        zIndex: 1
      }"
      @mousedown="startDrag($event, index)"
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
  background-color: green;
  border-radius: 50%;
  cursor: grab;
}

.circle2 {
  background-color: red;
}

.addButton {
  margin-right: 20px;
}
</style>
