<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import telefonoImg from '../assets/telefono.png'

const rootEl = ref<HTMLElement | null>(null)
const started = ref(false)
const typed = ref('')

// same technique as Hero's title: literal spaces baked into the string,
// rendered with white-space: pre + a hidden ghost that reserves the box
const full = '¿QUIERES                UNA\n         PÁGINA     ASÍ\n   DE                COOL?'

// DVD-screensaver bounce for the background contact text (same technique as About.vue)
const dvdEl = ref<HTMLElement | null>(null)
let rafId = 0
function startDvd() {
  const el = dvdEl.value
  const bounds = rootEl.value
  if (!el || !bounds) return
  let x = 40
  let y = 40
  let dx = 1.5
  let dy = 1.15
  const step = () => {
    // bounce within the CTA section itself (not the viewport) — the section
    // scrolls with the page, so its own box is the right frame of reference
    const cw = bounds.clientWidth
    const ch = bounds.clientHeight
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

function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms))
}
async function typewrite(text: string, target: (v: string) => void, speed = 55) {
  for (let i = 0; i < text.length; i++) {
    target(text.slice(0, i + 1))
    const ch = text[i]
    await sleep(ch === ' ' || ch === '\n' ? 12 : speed)
  }
}
async function run() {
  if (started.value) return
  started.value = true
  await typewrite(full, (v) => (typed.value = v), 55)
}

const textEl = ref<HTMLElement | null>(null)

// same reasoning as the Hero title: white-space: pre can't wrap, so its width
// is dictated by the loaded font's metrics on whatever screen this is running
// on. Measure and scale down only by the amount it actually overflows.
function fitText() {
  const el = textEl.value
  const host = rootEl.value
  if (!el || !host) return
  el.style.fontSize = ''
  const cs = getComputedStyle(host)
  const available =
    host.clientWidth - parseFloat(cs.paddingLeft) - parseFloat(cs.paddingRight)
  // the ghost span, not the <h2> — the heading is a flex item and its box
  // stops at the container edge while the pre-formatted text runs past it, so
  // measuring the heading always reports a comfortable fit
  const measured = el.querySelector<HTMLElement>('.cta__ghost') ?? el
  const natural = measured.getBoundingClientRect().width
  if (!available || !natural || natural <= available) return
  const size = parseFloat(getComputedStyle(el).fontSize)
  el.style.fontSize = `${(size * available * 0.995) / natural}px`
}

let fitRaf = 0
function scheduleFit() {
  cancelAnimationFrame(fitRaf)
  fitRaf = requestAnimationFrame(fitText)
}

onMounted(async () => {
  fitText()
  window.addEventListener('resize', scheduleFit)
  window.addEventListener('orientationchange', scheduleFit)
  try {
    await (document as unknown as { fonts: FontFaceSet }).fonts.ready
    fitText()
  } catch {
    /* ignore */
  }
})
onBeforeUnmount(() => {
  window.removeEventListener('resize', scheduleFit)
  window.removeEventListener('orientationchange', scheduleFit)
  cancelAnimationFrame(fitRaf)
})

let observer: IntersectionObserver | null = null
onMounted(() => {
  observer = new IntersectionObserver(
    (entries) => {
      for (const e of entries) {
        if (e.isIntersecting) {
          run()
          observer?.disconnect()
        }
      }
    },
    { threshold: 0.4 },
  )
  if (rootEl.value) observer.observe(rootEl.value)
})
onBeforeUnmount(() => observer?.disconnect())
</script>

<template>
  <section id="contacto" ref="rootEl" class="cta">
    <div ref="dvdEl" class="cta__dvd">ESCRÍBEME<br />gilbrtttt@gmail.com<br />@gilbrtttt</div>

    <img :src="telefonoImg" class="cta__phone" alt="" draggable="false" />

    <span class="cta__corner cta__corner--tl" />
    <span class="cta__corner cta__corner--tr" />
    <span class="cta__corner cta__corner--br" />
    <span class="cta__corner cta__corner--bl" />

    <h2 ref="textEl" class="cta__text">
      <span class="cta__ghost" aria-hidden="true">{{ full }}</span>
      <span class="cta__typed">{{ typed }}<span v-if="started" class="cta__caret">|</span></span>
    </h2>

    <footer class="cta__footer">
      Diseño y desarrollo por Gilberto Moncada. 2026
      <span class="cta__counter-block">
        <span class="cta__counter-sentence">
          Eres la persona número
          <a
            class="cta__counter"
            href="https://www.hitwebcounter.com/split-pdf"
            target="_blank"
            rel="noopener"
            title="Split PDF File Tool"
          >
            <img
              src="https://www.hitwebcounter.com/counter/counter.php?page=21512294&style=0006&nbdigits=5&type=ip"
              alt="Split PDF File Tool"
              decoding="async"
            />
          </a>
          en visitar esta página
        </span>
        <a
          class="cta__counter-attr"
          href="https://www.hitwebcounter.com/"
          target="_blank"
          rel="noopener"
        >www.hitwebcounter.com</a>
      </span>
    </footer>
  </section>
</template>

<style scoped>
@property --blink {
  syntax: '<number>';
  inherits: true;
  initial-value: 1;
}
@keyframes cta-blink-clock {
  50% { --blink: 0; }
}

.cta {
  position: relative;
  min-height: 100svh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(20px, 6vh, 80px) clamp(20px, 5vw, 64px);
  box-sizing: border-box;
  animation: cta-blink-clock 0.9s steps(1) infinite;
}

.cta__dvd {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 0;
  color: #000000;
  font-family: 'DotGothic16', monospace;
  font-size: clamp(20px, 4.2vw, 54px);
  line-height: 1.3;
  text-align: center;
  white-space: nowrap;
  pointer-events: none;
  user-select: none;
  will-change: transform;
}

.cta__phone {
  position: absolute;
  left: 50%;
  top: clamp(8px, 2.0vw, 15px);
  transform: translate(-50%, 0);
  width: min(11vw, 100px);
  height: auto;
  z-index: 1;
  image-rendering: pixelated;
  filter: blur(1.5px);
  pointer-events: none;
}

.cta__corner {
  position: absolute;
  width: 6px;
  height: 6px;
  background: #ff3d00;
  /* above the star */
  z-index: 2;
  opacity: var(--blink);
}
.cta__corner--tl { left: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.cta__corner--tr { right: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.cta__corner--br { right: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }
.cta__corner--bl { left: clamp(12px, 2.5vw, 32px); bottom: clamp(12px, 2.5vw, 32px); }

.cta__text {
  position: relative;
  /* above the star */
  z-index: 2;
  margin: 0;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(32px, 6.8vw, 104px);
  line-height: 1.25;
  letter-spacing: -0.02em;
  white-space: pre;
  text-align: left;
}
/* ghost reserves the full text box so the layout never shifts while typing */
.cta__ghost { visibility: hidden; }
.cta__typed { position: absolute; inset: 0; }
.cta__caret { opacity: var(--blink); }

.cta__footer {
  position: absolute;
  left: 50%;
  bottom: clamp(12px, 2.5vw, 32px);
  transform: translateX(-50%);
  z-index: 2;
  margin: 0;
  width: 100%;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(11px, 1.4vw, 16px);
  text-align: center;
}

/* third-party widget: it ships its own inline styles (border, height), so it
   can only be tamed from the outside. Sat below the credit line, dimmed at a
   fixed 0.55 so a visit counter doesn't compete with the page's own palette. */
.cta__counter-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: clamp(3px, 0.6vw, 5px);
  margin-top: clamp(6px, 1.2vw, 10px);
  opacity: 0.55;
}
/* the sentence and the counter image share one baseline, so the number reads
   as part of the phrase rather than as a separate badge underneath it */
.cta__counter-sentence {
  display: inline-flex;
  align-items: center;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.4em;
}
.cta__counter {
  display: inline-flex;
  line-height: 0;
}
.cta__counter img {
  display: block;
  height: clamp(14px, 2vw, 18px);
  width: auto;
  image-rendering: pixelated;
}
.cta__counter-attr {
  color: inherit;
  font-size: 0.8em;
  text-decoration: underline;
}

/* mobile: the tagline's spacing is baked into the string and sized for a
   wide screen — at the desktop scale its longest line is wider than a phone,
   which is one of the things pushing the page sideways. Scaling it down
   keeps the exact same shape, inside the screen. */
@media (max-width: 760px) {
  .cta {
    overflow: hidden;
    padding: clamp(24px, 8vh, 64px) clamp(14px, 4vw, 28px);
  }
  /* 27 characters on the longest line at DotGothic16's ~0.6em advance — the
     desktop 6.8vw would be about 110% of a phone's width. fitText() still
     measures and trims further if this device needs it. */
  .cta__text { font-size: clamp(20px, 7vw, 55px); }
  .cta__dvd { font-size: clamp(17px, 5vw, 30px); }
  .cta__phone {
    top: clamp(10px, 3vw, 20px);
    width: min(24vw, 120px);
  }
  .cta__footer { font-size: clamp(10px, 3vw, 13px); }
}
</style>
