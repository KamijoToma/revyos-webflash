<script setup lang="ts">
import FastBootFlash from './components/FastBootFlash.vue';
</script>

<template>
  <main class="download-section">
      <FastBootFlash />
  </main>
</template>

<style scoped>

.download-section {
  max-width: 90%; /* Adjust to fit the screen size */
  margin: 0 auto;
  padding: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh; /* Full viewport height */
  box-sizing: border-box; /* Ensure padding is included in the size */
}
</style>
