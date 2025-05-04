<script setup>
import { ref, computed } from "vue";

const RADIUS = 80;
const dragging = ref(null);

const circle1 = ref({ x: 50, y: 150 });

// GREEN CIRCLES
const green_count = ref(1);
const green_circles = ref([{ x: 50, y: 150 }]);

function removeGreenCircle() {
  green_count.value--;
  green_circles.value.pop();
}

function addGreenCircle() {
  green_count.value++;
  let new_circle = JSON.parse(JSON.stringify(green_circles.value.at(-1)));
  new_circle.y += 100;
  green_circles.value.push(new_circle);
}

const disableGreenAddButton = computed(() => {
  return !(green_count.value < 4);
});

const disableGreenDeleteButton = computed(() => {
  return !(green_count.value > 1);
});

// RED CIRCLE (ADD/REMOVE LOGIC)
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

const disableRedAddButton = computed(() => {
  return !(red_count.value < 4);
});
const disableRedDeleteButton = computed(() => {
  return !(red_count.value > 1);
});

//DRAGGING LOGIC
function startDrag(event, id) {
  const circle =
    id < 0 ? green_circles.value[-(id + 1)] : red_circles.value[id];
  const offsetX = event.clientX - circle.x;
  const offsetY = event.clientY - circle.y;

  dragging.value = { id, offsetX, offsetY };
}

function onMouseMove(event) {
  if (!dragging.value) return;

  const { id, offsetX, offsetY } = dragging.value;
  const newX = event.clientX - offsetX;
  const newY = event.clientY - offsetY;

  if (id < 0) {
    green_circles.value[-(id + 1)] = { x: newX, y: newY };
  } else {
    red_circles.value[id] = { x: newX, y: newY };
  }
}

function stopDrag() {
  dragging.value = null;
}

// touching logic
const isTouching = computed(() => {
  const r = RADIUS / 2;

  //green circle

  //red circles
  for (let gc of green_circles.value) {
    const center_x_1 = gc.x + r;
    const center_y_1 = gc.y + r;
    for (let rc of red_circles.value) {
      const center_x_2 = rc.x + r;
      const center_y_2 = rc.y + r;

      const dx = center_x_1 - center_x_2;
      const dy = center_y_1 - center_y_2;
      const distance = Math.sqrt(dx * dx + dy * dy);
      if (distance < RADIUS) {
        return true;
      }
    }
  }
  return false;
});
</script>

<template>
  <div id="app" @mousemove="onMouseMove" @mouseup="stopDrag">
    <div class="indicator">
      <div style="margin-bottom: 10px">
        Green is {{ isTouching ? "touching" : "not touching" }} a red circle
      </div>
      <div>
        <button
          :disabled="disableRedAddButton"
          class="addButton"
          @click="addRedCircle()"
        >
          Add Red Circle
        </button>
        <button :disabled="disableRedDeleteButton" @click="removeRedCircle()">
          Remove Red Circle
        </button>
      </div>
      <div>
        <button
          :disabled="disableGreenAddButton"
          class="addButton"
          @click="addGreenCircle()"
        >
          Add Green Circle
        </button>
        <button
          :disabled="disableGreenDeleteButton"
          @click="removeGreenCircle()"
        >
          Remove Green Circle
        </button>
      </div>
    </div>
    <div
      class="circle"
      v-for="(gcirc, index) in green_circles"
      :style="{ left: gcirc.x + 'px', top: gcirc.y + 'px', zIndex: 1 }"
      @mousedown="startDrag($event, -1 * index - 1)"
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
