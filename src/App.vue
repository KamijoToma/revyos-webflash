<script setup lang="ts">
import FastBootFlash from './components/FastBootFlash.vue';
</script>

<template>
  <header>
    <div class="wrapper">
    </div>
  </header>

  <main>
    <div class="download-section">
      <FastBootFlash />
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.download-section {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
