<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import IntroSequence from './components/IntroSequence.vue'
import Hero from './components/Hero.vue'
import About from './components/About.vue'
import Projects from './components/Projects.vue'
import CTA from './components/CTA.vue'

const introDone = ref(false)
const heroDone = ref(false)

// scroll drives the reveal: 0 = closed (small box in center), 1 = fully open
const progress = ref(0)
const scrollY = ref(0)
function onScroll() {
  const vh = window.innerHeight || 1
  scrollY.value = window.scrollY
  progress.value = Math.min(1, Math.max(0, window.scrollY / vh))
}
const aboutActive = computed(() => progress.value > 0.3)

// once fully open, the About rides up with the scroll so Projects replaces it
const leaveOffset = computed(() => {
  const vh = window.innerHeight || 1
  return Math.max(0, scrollY.value - vh)
})

// keep the page unscrollable until the hero has finished typing, so the reveal
// can only begin (from zero) once the text is loaded — never mid-way
function setScrollLock(lock: boolean) {
  document.documentElement.style.overflow = lock ? 'hidden' : ''
}
watch(introDone, (v) => {
  if (v && !heroDone.value) setScrollLock(true)
})
watch(heroDone, (v) => {
  if (v) setScrollLock(false)
})

onMounted(() => {
  // always start the flow from the top so the reveal plays from zero
  // (browsers otherwise restore the previous scroll position on reload)
  if ('scrollRestoration' in history) history.scrollRestoration = 'manual'
  window.scrollTo(0, 0)
  progress.value = 0
  window.addEventListener('scroll', onScroll, { passive: true })
})
onUnmounted(() => window.removeEventListener('scroll', onScroll))
</script>

<template>
  <IntroSequence v-if="!introDone" @done="introDone = true" />

  <div class="page">
    <div class="page__hero">
      <Hero :ready="introDone" @done="heroDone = true" />
    </div>

    <!-- once the box is open, this hides the hero behind so a fast-scroll gap
         between About and Projects never flashes the hero -->
    <div v-if="progress >= 0.99" class="page__backdrop" />


    <!-- reveal: hidden until the hero finished typing AND the user starts scrolling -->
    <div
      v-if="heroDone && progress > 0"
      class="reveal"
      :class="{ 'is-open': progress >= 0.999 }"
      :style="{ '--p': progress, transform: `translateY(${-leaveOffset}px)` }"
    >
      <div class="reveal__about">
        <About :active="aboutActive" />
      </div>
      <div class="reveal__frame" />
    </div>

    <!-- scroll room for the box to finish opening before Projects arrives -->
    <div class="reveal-spacer" />

    <!-- Projects scrolls up over the opened About -->
    <div class="projects-wrap">
      <Projects />
      <CTA />
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html {
  overflow-x: hidden;
  overflow-y: auto;
  scrollbar-width: none;
}
html::-webkit-scrollbar {
  display: none;
}
body {
  overscroll-behavior: none;
}

.page {
  position: relative;
}
.page__hero {
  position: sticky;
  top: 0;
  height: 100svh;
  z-index: 0;
}
.page__backdrop {
  position: fixed;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  background-color: #ffffff;
  background-image:
    linear-gradient(to right, rgba(0, 0, 0, 0.13) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0, 0, 0, 0.13) 1px, transparent 1px);
  background-size: clamp(15px, 2vw, 28px) clamp(15px, 2vw, 28px);
}
/* one viewport of scroll for the reveal to open before Projects comes up */
.reveal-spacer {
  height: 100svh;
}
/* Projects sits above the fixed About and scrolls up over it.
   Background lives here once so Projects/CTA don't each need their own. */
.projects-wrap {
  position: relative;
  z-index: 6;
  background-color: #ffffff;
  background-image:
    linear-gradient(to right, rgba(0, 0, 0, 0.13) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0, 0, 0, 0.13) 1px, transparent 1px);
  background-size: clamp(15px, 2vw, 28px) clamp(15px, 2vw, 28px);
}

.reveal {
  position: fixed;
  inset: 0;
  z-index: 5;
  pointer-events: none;
  animation: reveal-in 0.25s ease-out;
}
@keyframes reveal-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* full-viewport About, clipped to a centered window that grows with scroll */
.reveal__about {
  position: fixed;
  inset: 0;
  clip-path: inset(
    calc(49.5% * (1 - var(--p))) calc(49.5% * (1 - var(--p)))
  );
}
.reveal.is-open .reveal__about {
  pointer-events: auto;
}

/* red border framing the growing window */
.reveal__frame {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: calc(1vw + 99vw * var(--p));
  height: calc(1svh + 99svh * var(--p));
  border: 2px solid #ff3d00;
  box-sizing: border-box;
  pointer-events: none;
}
.reveal.is-open .reveal__frame {
  opacity: 0;
}
</style>
