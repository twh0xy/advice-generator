<script setup lang="ts">
import Card from './components/AdviceCard/Card.vue'
</script>

<template>
  <div class="app__card">
    <Card />
  </div>
</template>

<style scoped>
.app__card {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
