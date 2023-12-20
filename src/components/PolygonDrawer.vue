<template>
  <div ref="main_ref" class="main-wrapper" @pointerdown="addNewPoint">
    <div ref="target_ref" class="target" :class="{ 'inside' : isInsidePolygon(vertices, point_rect) }" />
    
    <svg ref="svg_ref" height="100%" width="100%" viewBox="0 0 100 100" preserveAspectRatio="none">
      <polygon :points="vPoints" fill="rgba(246, 99, 46, 0.5)" stroke="rgb(246, 99, 46)" stroke-width="0.4" />
    </svg>
    <div
      v-for="v, i in vertices"
      :key="i"
      class="vertex"
      :class="{ 'hover' : active_vertex == i }"
      :style="{ 'left': `${ v.x }%`, 'top': `${ v.y }%` }"
      @mouseleave="onPointerLeave"
      @mouseup="reset"
      @pointerup="reset"
      @pointerdown="setActivePoint(i)"
    />
  </div>
</template>

<script setup>
//=============================
// Imports
//=============================
import {
  ref,
  computed,
  onMounted,
  reactive,
  onUnmounted,
} from 'vue';


//=============================
// Consts
//=============================
const main_ref      = ref( undefined );
const svg_ref       = ref( undefined );
const vertices      = ref( [] );
const active_vertex = ref( -1 );
const main_rect     = ref( {} );
const point_rect    = ref( {} );
const target_ref = ref( undefined );

const pointer = reactive({
  down: false,
});

const vPoints = computed( () => vertices.value.map(v => `${v.x},${v.y}`).join(' '));

//=============================
// Function
//=============================
function isInsidePolygon( vertices, point ) {
  if ( vertices.length == 0 ) { return false; }
  let isInside = false;
  const _point = {
    x: (point.x - main_rect.value.x) * 100 / main_rect.value.width,
    y: (point.y - main_rect.value.y) * 100 / main_rect.value.height,
  };

  // consider a rect built around the polygon and check if the point is outside the perimeter
  let minX = vertices[0].x;
  let maxX = vertices[0].x;
  let minY = vertices[0].y;
  let maxY = vertices[0].y;

  for (let n = 1; n < vertices.length; n++) {
    let current_vertex = vertices[n];
    // update values if necessary
    minX = Math.min(current_vertex.x, minX);
    maxX = Math.max(current_vertex.x, maxX);
    minY = Math.min(current_vertex.y, minY);
    maxY = Math.max(current_vertex.y, maxY);
  }

  if (_point.x < minX || _point.x > maxX || _point.y < minY || _point.y > maxY) {
    // the point is obviously outside the polygon if is outside the rect built around the polygon
    return isInside;
  }

  let i = 0;
  let j = vertices.length - 1;

  for (i, j; i < vertices.length; j = i++) {
    let current_i = vertices[i];
    let current_j = vertices[j];
    // verify intersection between a line and a side of the polygon
    if ( (current_i.y > _point.y) != (current_j.y > _point.y) &&
         _point.x < (current_j.x - current_i.x) * (_point.y - current_i.y) / (current_j.y - current_i.y) + current_i.x ) {
      isInside = !isInside;
    }
  }
  return isInside;
}

function addNewPoint( e ) {
  if ( active_vertex.value == -1 ) {
    const { x, y } = getPercFromClient( e );
    vertices.value.push({ x, y });
  }
}

function setActivePoint(i) {
  pointer.down = true;
  active_vertex.value = i;
}


function onPointerMove( e ) {
  if ( active_vertex.value > -1 && pointer.down ) {
    const { x, y } = getPercFromClient( e );
    vertices.value[ active_vertex.value ] = { x, y };
  }
}

function getPercFromClient( e ) {
  return {
    x: (e.clientX - main_rect.value.x) * 100 / main_rect.value.width,
    y: (e.clientY - main_rect.value.y) * 100 / main_rect.value.height,
  };
}

function deletePoint() {
  if ( active_vertex.value > -1 ) {
    vertices.value.splice( active_vertex.value, 1);
    pointer.down = false;
    active_vertex.value = -1;
  }
}

function reset() {
  active_vertex.value = -1;
  pointer.down = false;
}

function onResize() {
  main_rect.value = main_ref.value.getBoundingClientRect();
  point_rect.value = target_ref.value.getBoundingClientRect();
}

//=============================
// Life cycle
//=============================
onMounted( () => {
  window.addEventListener('mousemove', onPointerMove);
  window.addEventListener('keydown', deletePoint);
  window.addEventListener('resize', onResize);
  onResize();
});

onUnmounted( () => {
  window.removeEventListener('mousemove', onPointerMove);
  window.removeEventListener('keydown', deletePoint);
  window.removeEventListener('resize', onResize);
})


</script>

<style lang="scss" scoped>
.main-wrapper {
  width: 100vw;
  height: 100vh;
  position: relative;
  background-color: #222;
  .vertex {
    position: absolute;
    transform: translate(-50%, -50%);
    width: 12px;
    height: 12px;
    border-radius: 50%;
    border: 1px solid rgba(0, 0, 0, 0.5);
    border-radius: 100%;
    background-color: rgb(246, 99, 46);
    touch-action: none;
    cursor: pointer;
    z-index: 0;
    &.hover {
      width: 30px;
      height: 30px;
      z-index: 1;
    }
  }
}

.target {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background-color: #ddd;
  top: 50%;
  left: 50%;
  position: absolute;
  transform: translate(-50%, -50%);
  &.inside {
    background-color: lightgreen;
  }
}
</style>