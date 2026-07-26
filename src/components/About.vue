<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import fishImg from '../assets/fish.png'
import catImg from '../assets/cat.png'
import ScratchCard from './ScratchCard.vue'

defineProps<{ active?: boolean }>()

// DVD-screensaver bounce for the background text
const dvdEl = ref<HTMLElement | null>(null)
let rafId = 0
function startDvd() {
  const el = dvdEl.value
  if (!el) return
  let x = 60
  let y = 60
  let dx = 1.5
  let dy = 1.15
  const step = () => {
    const cw = window.innerWidth
    const ch = window.innerHeight
    const ew = el.offsetWidth
    const eh = el.offsetHeight
    x += dx
    y += dy
    if (x <= 0) { x = 0; dx = -dx } else if (x + ew >= cw) { x = cw - ew; dx = -dx }
    if (y <= 0) { y = 0; dy = -dy } else if (y + eh >= ch) { y = ch - eh; dy = -dy }
    el.style.transform = `translate(${x}px, ${y}px)`
    rafId = requestAnimationFrame(step)
  }
  rafId = requestAnimationFrame(step)
}
onMounted(startDvd)
onBeforeUnmount(() => cancelAnimationFrame(rafId))

// typewriter for the scratch-cards' texts (same technique as Hero/CTA)
const bioFull = 'Soy Gilberto Moncada, estudiante de 6to semestre de ingeniería informática. Estoy iniciando en el desarrollo web.'
const bioText = ref('')
const bioStarted = ref(false)

const bio2Full = 'Ese pez me da mucha risa. Me gusta crear y por eso hice esta página, mi idea fue plasmar mi personalidad acá y creo que salió bien.'
const bio2Text = ref('')
const bio2Started = ref(false)

const bio3Full = '¿Tú quién eres? A mí me costó más escribir esto que hacer la página. Me gusta el naranja y mi top 4 de Letterboxd es Uncut Gems, Cars, Marriage Story y Obsession, y que paso? ah?.'
const bio3Text = ref('')
const bio3Started = ref(false)

function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms))
}
async function typewrite(full: string, target: (v: string) => void, speed = 25) {
  for (let i = 0; i < full.length; i++) {
    target(full.slice(0, i + 1))
    const ch = full[i]
    await sleep(ch === ' ' ? 8 : speed)
  }
}
onMounted(async () => {
  bioStarted.value = true
  await typewrite(bioFull, (v) => (bioText.value = v))
  await sleep(200)
  // shown immediately, no typewriter effect for this one
  bio2Started.value = true
  bio2Text.value = bio2Full
  await sleep(200)
  // shown immediately, no typewriter effect for this one
  bio3Started.value = true
  bio3Text.value = bio3Full
})

// LLAMA AHORA badge: 19-point star, inner ratio 0.73
const badgePoints = (() => {
  const cx = 100, cy = 100, outer = 100, inner = 73, rays = 19
  const pts: string[] = []
  for (let i = 0; i < rays * 2; i++) {
    const r = i % 2 === 0 ? outer : inner
    const a = (Math.PI / rays) * i - Math.PI / 2
    pts.push(`${(cx + r * Math.cos(a)).toFixed(2)},${(cy + r * Math.sin(a)).toFixed(2)}`)
  }
  return pts.join(' ')
})()
</script>

<template>
  <section class="about">
    <div ref="dvdEl" class="about__dvd">who the f*ck am i?</div>

    <span class="about__corner about__corner--tl" />
    <span class="about__corner about__corner--tr" />
    <span class="about__corner about__corner--br" />
    <span class="about__corner about__corner--bl" />

    <img :src="catImg" class="about__cat" alt="" draggable="false" />

    <h2 class="about__title">sobre mí<span class="about__caret">|</span></h2>

    <div class="about__777">777</div>
    <img :src="fishImg" class="about__fish" alt="" draggable="false" />

    <div class="about__badge" aria-hidden="true">
      <svg class="about__badge-star" viewBox="0 0 200 200" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
        <polygon :points="badgePoints" fill="#FF0000" />
      </svg>
      <span class="about__badge-text">QUIEN<br />ERES???</span>
    </div>

    <div class="about__box about__box--tr">
      <ScratchCard>
        <p class="about__box-text">{{ bioText }}<span v-if="bioStarted && bioText.length < bioFull.length" class="about__box-caret">|</span></p>
      </ScratchCard>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>
    <div class="about__box about__box--bl">
      <ScratchCard>
        <p class="about__box-text">{{ bio3Text }}<span v-if="bio3Started && bio3Text.length < bio3Full.length" class="about__box-caret">|</span></p>
      </ScratchCard>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>
    <div class="about__box about__box--br">
      <ScratchCard>
        <p class="about__box-text">{{ bio2Text }}<span v-if="bio2Started && bio2Text.length < bio2Full.length" class="about__box-caret">|</span></p>
      </ScratchCard>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>
  </section>
</template>

<style scoped>
/* shared blink clock (drives --blink for caret + corners) */
@property --blink {
  syntax: '<number>';
  inherits: true;
  initial-value: 1;
}
@keyframes about-blink {
  50% { --blink: 0; }
}
@keyframes spin-left {
  to { transform: rotate(-360deg); }
}
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-14px); }
}
@keyframes swim-diagonal {
  0%, 100% { transform: translate(0, 0); }
  50% { transform: translate(18px, -18px); }
}
@keyframes pulse-scale {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.12); }
}

.about {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 100svh;
  background-color: #ffffff;
  background-image:
    linear-gradient(to right, rgba(0, 0, 0, 0.13) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0, 0, 0, 0.13) 1px, transparent 1px);
  background-size: clamp(15px, 2vw, 28px) clamp(15px, 2vw, 28px);
  overflow: hidden;
  animation: about-blink 0.9s steps(1) infinite;
}

/* bouncing background text (DVD-screensaver style) */
.about__dvd {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 0;
  color: #000000;
  font-family: 'DotGothic16', monospace;
  font-size: clamp(22px, 5vw, 72px);
  white-space: nowrap;
  pointer-events: none;
  user-select: none;
  will-change: transform;
}

/* orange frame corners at the viewport edges, like the hero */
.about__corner {
  position: absolute;
  width: 6px;
  height: 6px;
  background: #ff3d00;
  opacity: var(--blink);
}
.about__corner--tl { left: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.about__corner--tr { right: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.about__corner--br { right: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }
.about__corner--bl { left: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }

.about__cat {
  position: absolute;
  left: 6vw;
  top: -4vh;
  width: min(32vw, 400px);
  height: auto;
  z-index: 0;
  image-rendering: pixelated;
  transform-origin: center;
  animation: pulse-scale 1.6s ease-in-out infinite;
}

.about__title {
  position: absolute;
  left: 50%;
  top: 46%;
  transform: translate(-50%, -50%);
  margin: 0;
  z-index: 1;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(34px, 9vw, 130px);
  line-height: 1;
  white-space: nowrap;
}
.about__caret {
  opacity: var(--blink);
}

.about__777 {
  position: absolute;
  left: 77%;
  top: 12%;
  z-index: 1;
  font-family: 'Faster One', cursive;
  font-size: clamp(30px, 5.6vw, 100px);
  color: #ff3d00;
  -webkit-text-stroke: 1.5px #00a50b;
  transform-origin: center;
  animation: spin-left 6s linear infinite;
}

.about__fish {
  position: absolute;
  left: 69%;
  top: 25%;
  width: min(70vw, 350px);
  height: auto;
  z-index: 1;
  image-rendering: pixelated;
  animation: swim-diagonal 2.8s ease-in-out infinite;
}

.about__badge {
  position: absolute;
  left: 40.5%;
  top: 62%;
  width: min(19vw, 240px);
  aspect-ratio: 80 / 79;
  z-index: 2;
}
.about__badge-star {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  transform-origin: center;
  animation: spin-left 12s linear infinite;
}
.about__badge-text {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  transform: rotate(-14deg);
  color: #ffffff;
  font-family: 'Black Han Sans', sans-serif;
  font-size: clamp(14px, 3.2vw, 32px);
  line-height: 1.05;
}

/* green content frames with white corner marks (aspect kept from Figma) */
.about__box {
  position: absolute;
  border: 2px solid #87ddff;
  z-index: 1;
}
.about__box--tr { left: 43%; top: 7%; width: min(27vw, 320px); aspect-ratio: 126 / 66; }
.about__box--bl { left: 12%; bottom: 7%; width: min(26vw, 310px); aspect-ratio: 108 / 69; }
.about__box--br { left: 65%; bottom: 9%; width: min(27vw, 320px); aspect-ratio: 126 / 63; }

.box-corner {
  position: absolute;
  width: 8px;
  height: 8px;
  background: #ffffff;
  border: 1px solid #111;
  box-sizing: border-box;
}
.box-corner--tl { left: -1px; top: -1px; transform: translate(-50%, -50%); }
.box-corner--tr { right: -1px; top: -1px; transform: translate(50%, -50%); }
.box-corner--br { right: -1px; bottom: -1px; transform: translate(50%, 50%); }
.box-corner--bl { left: -1px; bottom: -1px; transform: translate(-50%, 50%); }

.about__box-text {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 10%;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  text-align: left;
  background: transparent;
  font-family: 'DotGothic16', monospace;
  font-weight: 550;
  color: #111111;
  font-size: clamp(11px, 1.3vw, 17px);
  line-height: 1.35;
}
.about__box-caret { opacity: var(--blink); }

/* mobile: the decorative cluster (title/cat/777/fish/badge) keeps its
   absolute positioning but pinned to viewport-height units so it stays
   within a fixed top zone; the 3 scratch boxes drop into normal document
   flow and stack below that zone, so nothing can ever overlap the text */
@media (max-width: 700px) {
  .about {
    height: auto;
    overflow: visible;
    padding-bottom: 24px;
  }

  .about__cat { left: 4vw; top: 1vh; width: 36vw; }
  .about__title { top: 21vh; font-size: clamp(30px, 11vw, 60px); }
  .about__777 { left: 68vw; top: 3vh; font-size: clamp(26px, 9vw, 50px); }
  .about__fish { left: 55vw; top: 11vh; width: 42vw; }
  .about__badge { left: 30vw; top: 33vh; width: 36vw; }

  .about__box {
    position: static;
    width: 88%;
    margin: 14px auto 0;
    aspect-ratio: 16 / 10;
  }
  /* first box in DOM order — pushes the whole stack below the decorative zone */
  .about__box--tr {
    margin-top: 56vh;
  }
}
</style>
