<template>
  <div class="top-main">
    <TetrisContainer/>
  </div>
</template>

<script setup>
import TetrisContainer from './components/TetrisContainer.vue'
</script>

<style scoped>
.top-main {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
