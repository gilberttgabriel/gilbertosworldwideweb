<script setup lang="ts">
import { ref } from 'vue'
import IntroSequence from './components/IntroSequence.vue'

const introDone = ref(false)
</script>

<template>
  <IntroSequence v-if="!introDone" @done="introDone = true" />

  <main class="hero">
    <h1>GILBERTO MONCADA</h1>
    <p>Desarrollador de Software</p>
  </main>
</template>

<style scoped>
.hero {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #0e0d0b;
  color: #f4f1ea;
  font-family: sans-serif;
}
</style>
