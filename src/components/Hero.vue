<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'

const props = defineProps<{ ready?: boolean }>()
const emit = defineEmits<{ done: [] }>()

const titleFull = 'GILBERTO’S\n      WORLDWIDE\nWEB        SPACE'
const tagEsFull = 'desarrollador creativo\necléctico\nme gusta crear'
const tagEnFull = 'creative developer\neclectic\nlet’s create'

const titleText = ref('')
const tagEsText = ref('')
const tagEnText = ref('')
const started = ref(false)
const tagsStarted = ref(false)
const showNav = ref(false)

const navFull = ['sobre mi', 'proyectos', 'contacto']
const navTexts = ref(['', '', ''])

function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms))
}

async function typewrite(full: string, target: (v: string) => void, speed = 55) {
  for (let i = 0; i < full.length; i++) {
    target(full.slice(0, i + 1))
    const ch = full[i]
    await sleep(ch === ' ' || ch === '\n' ? 12 : speed)
  }
}

async function run() {
  started.value = true
  await typewrite(titleFull, (v) => (titleText.value = v), 60)
  await sleep(250)
  tagsStarted.value = true
  await Promise.all([
    typewrite(tagEsFull, (v) => (tagEsText.value = v), 30),
    typewrite(tagEnFull, (v) => (tagEnText.value = v), 30),
  ])
  await sleep(200)
  showNav.value = true
  await nextTick()
  for (let i = 0; i < navFull.length; i++) {
    await typewrite(navFull[i] ?? '', (v) => (navTexts.value[i] = v), 40)
  }
  emit('done')
}

watch(
  () => props.ready,
  (v) => {
    if (v && !started.value) run()
  },
  { immediate: true },
)

// scrolls to y and resolves once the browser's smooth-scroll has actually
// settled there (not just "animation started"), so a second scroll queued
// right after doesn't get cut short or skipped
function scrollToY(y: number) {
  return new Promise<void>((resolve) => {
    window.scrollTo({ top: y, behavior: 'smooth' })
    if ('onscrollend' in window) {
      const onEnd = () => {
        window.removeEventListener('scrollend', onEnd)
        resolve()
      }
      window.addEventListener('scrollend', onEnd, { once: true })
    } else {
      setTimeout(resolve, 700)
    }
  })
}

// every link plays the full Hero → About reveal first (the rectangle
// growing from the center), then — for "proyectos"/"contacto" — continues
// scrolling on to that section once the reveal has fully opened
async function handleNavClick(item: string) {
  await scrollToY(window.innerHeight)
  if (item === 'sobre mi') return
  const id = item === 'proyectos' ? 'proyectos' : item === 'contacto' ? 'contacto' : null
  const el = id ? document.getElementById(id) : null
  if (el) await scrollToY(el.getBoundingClientRect().top + window.scrollY)
}
</script>

<template>
  <section class="hero" :class="{ 'is-ready': ready }">
    <div v-if="started" class="hero__grid" aria-hidden="true" />

    <span class="hero__corner hero__corner--tl" />
    <span class="hero__corner hero__corner--tr" />
    <span class="hero__corner hero__corner--br" />
    <span class="hero__corner hero__corner--bl" />

    <div class="hero__content">
      <svg
        v-if="started"
        class="hero__star"
        viewBox="0 0 256 175"
        preserveAspectRatio="none"
        xmlns="http://www.w3.org/2000/svg"
        aria-hidden="true"
      >
        <path
          d="M128 0L143.185 62.4398L218.51 25.6282L164.66 77.1197L256 87.5L164.66 97.8803L218.51 149.372L143.185 112.56L128 175L112.815 112.56L37.4903 149.372L91.3405 97.8803L0 87.5L91.3405 77.1197L37.4903 25.6282L112.815 62.4398L128 0Z"
          fill="#F8E10E"
        />
      </svg>

      <h1 class="hero__title">
        <span class="hero__ghost" aria-hidden="true">{{ titleFull }}</span>
        <span class="hero__typed">{{ titleText }}<span v-if="started" class="caret">|</span></span>
      </h1>
    </div>

    <div v-if="tagsStarted" class="hero__tagframe hero__tagframe--es">
      <span class="frame-corner frame-corner--tl" />
      <span class="frame-corner frame-corner--tr" />
      <span class="frame-corner frame-corner--br" />
      <span class="frame-corner frame-corner--bl" />
      <p class="hero__tag">{{ tagEsText }}<span class="caret">|</span></p>
    </div>

    <div v-if="tagsStarted" class="hero__tagframe hero__tagframe--en">
      <span class="frame-corner frame-corner--tl" />
      <span class="frame-corner frame-corner--tr" />
      <span class="frame-corner frame-corner--br" />
      <span class="frame-corner frame-corner--bl" />
      <p class="hero__tag">{{ tagEnText }}<span class="caret">|</span></p>
    </div>

    <nav v-if="showNav" class="hero__tagframe hero__nav">
      <span class="frame-corner frame-corner--tl" />
      <span class="frame-corner frame-corner--tr" />
      <span class="frame-corner frame-corner--br" />
      <span class="frame-corner frame-corner--bl" />
      <ul class="hero__navlist">
        <li v-for="(item, i) in navFull" :key="item">
          <a
            class="hero__navlink"
            href="#"
            @click.prevent="handleNavClick(item)"
          ><span class="hero__navslash">/</span>{{ navTexts[i] }}<span v-if="i === navFull.length - 1" class="caret">|</span></a>
        </li>
      </ul>
    </nav>
  </section>
</template>

<style scoped>
/* single shared blink clock: one animation drives --blink for every blinking
   element, so carets and corner marks all titilate in sync */
@property --blink {
  syntax: '<number>';
  inherits: true;
  initial-value: 1;
}

.hero {
  position: relative;
  height: 100%;
  min-height: 100svh;
  background-color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  padding: clamp(24px, 8vh, 96px) clamp(16px, 6vw, 64px);
  box-sizing: border-box;
  animation: blink-clock 0.9s steps(1) infinite;
}
@keyframes blink-clock {
  50% { --blink: 0; }
}

/* small grid that boots up from the center outward, Tron-style */
.hero__grid {
  position: absolute;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  background-image:
    linear-gradient(to right, rgba(0, 0, 0, 0.13) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0, 0, 0, 0.13) 1px, transparent 1px);
  background-size: clamp(15px, 2vw, 28px) clamp(15px, 2vw, 28px);
  clip-path: circle(0% at 50% 55%);
  animation: grid-boot 1.1s cubic-bezier(0.35, 0, 0.2, 1) forwards;
}
@keyframes grid-boot {
  0% { clip-path: circle(0% at 50% 55%); opacity: 0.2; }
  12% { opacity: 1; }
  100% { clip-path: circle(140% at 50% 55%); opacity: 1; }
}

.hero__corner {
  position: absolute;
  width: 5px;
  height: 5px;
  background: #ff3d00;
  /* stay hidden until the intro's transition dots have landed on the corners */
  opacity: 0;
}
.is-ready .hero__corner {
  opacity: var(--blink);
}
.hero__corner--tl { left: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.hero__corner--tr { right: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.hero__corner--br { right: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }
.hero__corner--bl { left: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }

/* yellow star burst behind the title — Figma frame 10 horizontal placement
   (center x at 65.4%), centered vertically on the title so it reads symmetric */
.hero__star {
  position: absolute;
  left: 65.4%;
  top: 57%;
  width: 54.7%;
  aspect-ratio: 256 / 175;
  z-index: 0;
  pointer-events: none;
  transform: translate(-50%, -50%);
  transform-origin: center;
  /* "matrix" render (clip-path) reveals it top→bottom, then it spins left slowly */
  clip-path: inset(0 0 100% 0);
  animation:
    star-matrix 0.9s steps(14) forwards,
    star-spin 30s linear infinite;
}
@keyframes star-matrix {
  to { clip-path: inset(0 0 0 0); }
}
@keyframes star-spin {
  from { transform: translate(-50%, -50%) rotate(0deg); }
  to { transform: translate(-50%, -50%) rotate(-360deg); }
}

.hero__content {
  position: relative;
  z-index: 1;
  width: fit-content;
  max-width: 100%;
  margin: 0 auto;
}

.hero__title {
  position: relative;
  z-index: 1;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  /* scales with viewport so the 3 lines span nearly edge-to-edge, as in Figma */
  font-size: clamp(28px, 10.6vw, 200px);
  line-height: 1.2;
  text-align: left;
  white-space: pre;
}
/* ghost reserves the full title box so the centered block never shifts while typing */
.hero__ghost {
  visibility: hidden;
}
.hero__typed {
  position: absolute;
  inset: 0;
}

/* framed tags (Figma frame 10): corner marks around each tagline */
.hero__tagframe {
  position: absolute;
  z-index: 2;
  padding: 12px 16px;
  /* light-blue framing line (Figma frame 10) */
  border: 1px solid #87ddff;
}
.hero__tagframe--es {
  right: 10%;
  top: 10%;
}
.hero__tagframe--en {
  left: 7%;
  top: 41%;
}

/* navigation menu, framed like the tags (Figma frame 10, bottom area) */
.hero__nav {
  left: 30%;
  bottom: 9%;
  padding: 20px 28px;
}
.hero__navlist {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.hero__navlink {
  display: inline-block;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  text-decoration: none;
  font-size: clamp(15px, 3.4vw, 30px);
  line-height: 1.4;
}
.hero__navslash {
  margin-right: 0.4em;
}
.hero__navlink:hover {
  text-decoration: underline;
  text-underline-offset: 3px;
}

.frame-corner {
  position: absolute;
  width: 6px;
  height: 6px;
  background: #ffffff;
  border: 1px solid #000000;
  box-sizing: border-box;
}
/* centered on each corner of the border box so the blue line runs through their middle */
.frame-corner--tl { left: -1px; top: -1px; transform: translate(-50%, -50%); }
.frame-corner--tr { right: -1px; top: -1px; transform: translate(50%, -50%); }
.frame-corner--br { right: -1px; bottom: -1px; transform: translate(50%, 50%); }
.frame-corner--bl { left: -1px; bottom: -1px; transform: translate(-50%, 50%); }

.hero__tag {
  margin: 0;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  /* stays ~0.21x the title (as in Figma) so the format scales the same on any screen */
  font-size: clamp(15px, 3.4vw, 30px);
  line-height: 1.5;
  white-space: pre;
}

.caret {
  opacity: var(--blink);
}

/* portrait: title centered with lines spread vertically; framed tags move to
   the top/bottom extremes so they use the height and never overlap the title */
@media (orientation: portrait) {
  .hero__title {
    /* spread the 3 lines out to use the extra vertical room on phones */
    line-height: 2.6;
  }
  .hero__tag {
    font-size: clamp(15px, 3.4vw, 30px);
  }
  .hero__tagframe--es {
    top: 5%;
    right: 8%;
  }
  .hero__tagframe--en {
    top: 42%;
    left: 5%;
  }
}
</style>
