<template>
  <h1>Ray casting algorithm</h1>
  <PolygonDrawer />
</template>

<script setup>
import PolygonDrawer from './components/PolygonDrawer.vue'
</script>


<style scoped>
h1 {
  position: absolute;
  top: 22px;
  left: 22px;
  font-family: 'Courier New', Courier, monospace;
  z-index: 2;
  color: #ccc;
  user-select: none;
  pointer-events: none;
  margin: 0;
  padding: 0;
}
</style>
