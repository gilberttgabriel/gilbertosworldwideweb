<script setup lang="ts">
import { ref, computed, watch, nextTick, onMounted, onUnmounted } from 'vue'

const props = defineProps<{ ready?: boolean }>()
const emit = defineEmits<{ done: [] }>()

// the phone is its own frame in Figma, not the desktop one squeezed: there the
// gaps baked into the title are tighter, so its longest line is 13 characters
// instead of 16 — which is exactly what lets it run edge to edge on a narrow
// screen at a size that still reads as the same composition. Those gaps are
// literal characters inside a white-space: pre string, so they're content and
// the breakpoint has to be read here; no stylesheet can reach them.
const NARROW = '(max-width: 760px)'
const narrow = ref(window.matchMedia(NARROW).matches)

const titleFull = computed(() =>
  narrow.value
    ? 'GILBERTO’S\n   WORLDWIDE\nWEB     SPACE'
    : 'GILBERTO’S\n      WORLDWIDE\nWEB        SPACE',
)
// the phone frame carries a single tagline box, and its last line is the
// English one — the two-column es/en split only exists on the wide layout
const tagEsFull = computed(() =>
  narrow.value
    ? 'desarrollador creativo\necléctico\nlet’s create'
    : 'desarrollador creativo\necléctico\nme gusta crear',
)
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
  await typewrite(titleFull.value, (v) => (titleText.value = v), 60)
  await sleep(250)
  tagsStarted.value = true
  await Promise.all([
    typewrite(tagEsFull.value, (v) => (tagEsText.value = v), 30),
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

const heroEl = ref<HTMLElement | null>(null)
const titleEl = ref<HTMLElement | null>(null)

// the title is white-space: pre, so it can never wrap — its width is however
// wide the loaded font renders those three lines, which no CSS clamp() can
// predict: the fallback the browser uses while DotGothic16 is still arriving
// is measurably wider, and every phone has a different viewport anyway. So
// measure the real thing against the room this screen actually has and pull
// the size down only by however much it overflows. On a screen where it
// already fits (desktop) this changes nothing.
function fitTitle() {
  const el = titleEl.value
  const host = heroEl.value
  if (!el || !host) return
  // drop any previous correction first, so we always measure the size the
  // stylesheet asked for rather than compounding our own adjustments
  el.style.fontSize = ''
  const cs = getComputedStyle(host)
  const available =
    host.clientWidth - parseFloat(cs.paddingLeft) - parseFloat(cs.paddingRight)
  // measure the ghost span, never the <h1>. .hero__content is capped at
  // max-width: 100%, so the heading's own box stops growing at the container
  // edge and the text spills out of it instead — meaning the heading always
  // measures as exactly "fits" no matter how far the text actually runs past
  // the screen. The ghost is inline, so its box is the real rendered width of
  // the longest line.
  const measured = el.querySelector<HTMLElement>('.hero__ghost') ?? el
  const natural = measured.getBoundingClientRect().width
  if (!available || !natural || natural <= available) return
  const size = parseFloat(getComputedStyle(el).fontSize)
  // the 0.995 absorbs sub-pixel rounding, which on a phone is enough to leave
  // the last character clipped
  el.style.fontSize = `${(size * available * 0.995) / natural}px`
}

let fitRaf = 0
function scheduleFit() {
  cancelAnimationFrame(fitRaf)
  fitRaf = requestAnimationFrame(fitTitle)
}

// rotating across the breakpoint swaps those strings underneath text that has
// already finished typing, which would leave the visible copy and the ghost
// that reserves its box disagreeing about how wide the block is. Re-sync only
// what is already complete: if the typewriter is still mid-word, leave it be
// rather than snapping the animation to the end.
const mql = window.matchMedia(NARROW)
function onMq(e: MediaQueryListEvent) {
  narrow.value = e.matches
}
watch(titleFull, (v, old) => {
  if (titleText.value === old) titleText.value = v
  scheduleFit()
})
watch(tagEsFull, (v, old) => {
  if (tagEsText.value === old) tagEsText.value = v
})

let ro: ResizeObserver | null = null

onMounted(async () => {
  mql.addEventListener('change', onMq)
  fitTitle()

  // watching the section itself catches every width change, including the
  // ones a phone makes without firing a resize event (URL bar collapsing,
  // rotation). Observing .hero and resizing only the title inside it means
  // this can't feed back into itself.
  if ('ResizeObserver' in window && heroEl.value) {
    ro = new ResizeObserver(scheduleFit)
    ro.observe(heroEl.value)
  }
  window.addEventListener('resize', scheduleFit)
  window.addEventListener('orientationchange', scheduleFit)

  // the webfont swapping in changes every measurement. fonts.ready alone
  // isn't enough — it resolves as soon as nothing is pending, which can be
  // before Google's stylesheet has even asked for the face — so name the
  // font explicitly and wait for that.
  try {
    const fonts = (document as unknown as { fonts: FontFaceSet }).fonts
    await fonts.load('1em DotGothic16')
    await fonts.ready
    fitTitle()
  } catch {
    /* ignore */
  }
})
onUnmounted(() => {
  ro?.disconnect()
  mql.removeEventListener('change', onMq)
  window.removeEventListener('resize', scheduleFit)
  window.removeEventListener('orientationchange', scheduleFit)
  cancelAnimationFrame(fitRaf)
})

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
  <section ref="heroEl" class="hero" :class="{ 'is-ready': ready }">
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

      <h1 ref="titleEl" class="hero__title">
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

/* mobile: its own composition, taken from the phone frame in Figma. The
   collage re-centres rather than restacking — the star grows to nearly the
   full width of the screen and burns *behind the middle* of the title instead
   of off to its right, the three title lines spread far apart so each one
   lands on a different part of the star, and only two framed boxes survive:
   the tagline pinned top-right and the nav low and left of centre.
   Everything here is written against the title's own em or against the
   viewport width — never against a device's pixel height — so the pieces hold
   their relationship to each other as one composition on any phone, instead
   of needing a set of numbers per screen size. */
@media (max-width: 760px) {
  .hero {
    /* the parent (.page__hero) is fixed to the viewport here, so the base
       height: 100% already measures exactly one screen — this just stops the
       svh floor from ever forcing it taller than what's actually visible */
    min-height: 0;
    /* the title is meant to reach for the edges, so the gutter shrinks to
       roughly what the frame leaves around it */
    padding: clamp(16px, 4vh, 48px) clamp(10px, 3vw, 24px);
  }

  /* 13 characters on the longest line at DotGothic16's measured 0.5em advance
     is 6.5em, which at 13.3vw comes to 86% of the screen — the edge-to-edge
     look of the frame, with the gutter left over. The svh term only bites on a
     short or landscape viewport: the block below stands 8.8em tall, so capping
     the em at 9svh keeps it on screen there rather than letting .hero clip it.
     fitTitle() still measures the real render and trims if a device disagrees. */
  .hero__title {
    font-size: min(13.3vw, 9svh);
    /* the lines sit deliberately far apart: GILBERTO'S clears the top of the
       star, WORLDWIDE crosses its middle and WEB SPACE lands on its bottom
       point. This single number is what holds that three-way alignment. */
    line-height: 2.94;
  }

  /* sized and placed against .hero__content — the title's own box — rather
     than the viewport, so the alignment above survives at any size: 114% of
     the title's width, centred a third of a line below WORLDWIDE. It runs
     past the right edge exactly as it does in the frame; .hero's
     overflow: hidden is what trims it. */
  .hero__star {
    left: 57.2%;
    top: 53.5%;
    width: 114.4%;
  }

  /* one tagline on this frame, hard into the top-right corner. It clears the
     corner mark horizontally (the mark sits further right than the frame's
     edge), so it can sit that high without colliding. */
  .hero__tagframe--es {
    right: 6%;
    top: 7%;
  }
  .hero__tagframe--en {
    display: none;
  }
  /* these are absolutely positioned, so they shrink-to-fit their own text —
     and that text is white-space: pre, which can't wrap. Left alone, a frame
     simply grows as wide as its longest line and walks off the screen. The
     cap plus pre-wrap makes that structurally impossible: it can't exceed the
     screen, and if a line ever would, it wraps instead of overflowing. */
  .hero__tagframe {
    max-width: 92%;
    padding: 6px 8px;
  }
  .hero__tag {
    white-space: pre-wrap;
    font-size: clamp(12px, 4.2vw, 22px);
    line-height: 1.35;
  }

  /* left of centre and low, under the last title line. The title block's box
     reaches further down than its glyphs do (half a 2.94 line-height of
     leading hangs below WEB SPACE), so these two overlap as boxes and never
     visually — and the nav's higher z-index keeps it clickable regardless. */
  .hero__nav {
    left: 29%;
    bottom: 8%;
    padding: 6px 8px;
  }
  .hero__navlist {
    gap: 0;
  }
  .hero__navlink {
    font-size: clamp(12px, 4.2vw, 22px);
    line-height: 1.35;
  }
}
</style>
