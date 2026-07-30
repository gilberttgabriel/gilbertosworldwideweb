<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import fishImg from '../assets/fish.png'
import catImg from '../assets/cat.png'
import ScratchCard from './ScratchCard.vue'

const props = defineProps<{ active?: boolean }>()

// the phone has its own frame in Figma, and there the scratch-off surface is
// gone: the two cards it keeps are simply legible. Swapping ScratchCard for a
// plain div — rather than hiding its canvas with CSS — means no canvas is ever
// allocated, painted pixel by pixel, or listening for pointer events on a
// phone, which is also where that per-pixel fill is slowest.
const NARROW = '(max-width: 760px)'
const mql = window.matchMedia(NARROW)
const narrow = ref(mql.matches)
function onMq(e: MediaQueryListEvent) {
  narrow.value = e.matches
}
onMounted(() => mql.addEventListener('change', onMq))
onBeforeUnmount(() => mql.removeEventListener('change', onMq))

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
const bioFull = 'Soy Gilberto Moncada, estudiante de 6to semestre de ingeniería informática. Estoy iniciando en el desarrollo web. Actualmente viviendo Caracas.'
const bioText = ref('')
const bioStarted = ref(false)

const bio2Full = 'Ese pez me da mucha risa. Me gusta crear y por eso hice esta página, mi idea fue plasmar mi personalidad acá y creo que salió bien.'
const bio2Text = ref('')
const bio2Started = ref(false)

const bio3Full = '¿Tú quién eres? A mí me costó más escribir esto que hacer la página. Me gusta el naranja y mi top 4 de Letterboxd es Uncut Gems, Cars, Marriage Story y Obsession.'
const bio3Text = ref('')
const bio3Started = ref(false)

// mobile only: the phone frame leaves the top-right corner empty, and an empty
// corner in a collage this busy reads as a mistake rather than as breathing room
const bio4Full = 'no sé qué poner aquí y me daba toc que esta esquina quedara vacía'
const bio4Text = ref('')
const bio4Started = ref(false)

// The cards typed back to back, driven by ONE requestAnimationFrame loop reading
// the wall clock, rather than by a chain of ~450 setTimeouts.
//
// rAF is tied to the compositor, so it keeps pace with what is actually on
// screen and stops cleanly when the page is hidden. And because the character
// count is derived from elapsed time rather than incremented once per callback,
// a dropped frame is caught up on the next one instead of falling permanently
// behind — the text lands on time whatever the phone was busy with, which a
// self-chaining timer can never promise.
const CHAR_MS = 25 // per character
const SPACE_MS = 8 // spaces go quicker, so words land as words

// how long into the run each character of a string has been typed
function schedule(full: string) {
  const at: number[] = []
  let t = 0
  for (const ch of full) {
    t += ch === ' ' ? SPACE_MS : CHAR_MS
    at.push(t)
  }
  return { at, total: t }
}
// all four at once, off the same clock: every card reads the same `elapsed`
// rather than waiting for the one before it to finish, so they run in parallel
// at the same per-character speed and the shortest simply lands first. On
// desktop the fourth card is display: none, and typing into a hidden element
// costs nothing.
const plan = [
  { ...schedule(bioFull), full: bioFull, text: bioText, started: bioStarted },
  { ...schedule(bio2Full), full: bio2Full, text: bio2Text, started: bio2Started },
  { ...schedule(bio3Full), full: bio3Full, text: bio3Text, started: bio3Started },
  { ...schedule(bio4Full), full: bio4Full, text: bio4Text, started: bio4Started },
]

let typeRaf = 0
let typeStart = 0
function typeFrame(now: number) {
  if (!typeStart) typeStart = now
  const elapsed = now - typeStart
  let done = true
  for (const card of plan) {
    card.started.value = true
    // binary-search-free: the schedule is short, and a linear scan of a
    // hundred-odd numbers once a frame is nothing next to a repaint
    let n = 0
    while (n < card.at.length && (card.at[n] as number) <= elapsed) n++
    card.text.value = card.full.slice(0, n)
    if (n < card.full.length) done = false
  }
  if (!done) typeRaf = requestAnimationFrame(typeFrame)
}

// start when the section is actually on screen rather than when it mounts: the
// reveal mounts this component the instant the box begins to open, so typing
// from mount means the first card is already part-written by the time there is
// anything to look at
watch(
  () => props.active,
  (v) => {
    if (v && !typeRaf) typeRaf = requestAnimationFrame(typeFrame)
  },
  { immediate: true },
)
onBeforeUnmount(() => cancelAnimationFrame(typeRaf))

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

    <!-- mobile only (display: none on desktop), so it needs no ScratchCard -->
    <div class="about__box about__box--tr2">
      <div class="about__card">
        <p class="about__box-text">{{ bio4Text }}<span v-if="bio4Started && bio4Text.length < bio4Full.length" class="about__box-caret">|</span></p>
      </div>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>

    <div class="about__box about__box--tr">
      <component :is="narrow ? 'div' : ScratchCard" class="about__card">
        <p class="about__box-text">{{ bioText }}<span v-if="bioStarted && bioText.length < bioFull.length" class="about__box-caret">|</span></p>
      </component>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>
    <div class="about__box about__box--bl">
      <component :is="narrow ? 'div' : ScratchCard" class="about__card">
        <p class="about__box-text">{{ bio3Text }}<span v-if="bio3Started && bio3Text.length < bio3Full.length" class="about__box-caret">|</span></p>
      </component>
      <span class="box-corner box-corner--tl" />
      <span class="box-corner box-corner--tr" />
      <span class="box-corner box-corner--br" />
      <span class="box-corner box-corner--bl" />
    </div>
    <div class="about__box about__box--br">
      <component :is="narrow ? 'div' : ScratchCard" class="about__card">
        <p class="about__box-text">{{ bio2Text }}<span v-if="bio2Started && bio2Text.length < bio2Full.length" class="about__box-caret">|</span></p>
      </component>
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
/* The two right-hand cards can't just take a width — each of them runs into
   something, and a plain min(vw, px) only happens to clear it at one viewport.
     --tr  starts at 43% and the fish's INK starts at 69% + 13.1% of the fish's
           own 350px box (measured off fish.png's alpha, not its frame), so the
           room it actually has is 26vw + 45.8px. 26vw + 38px keeps ~8px of air
           at every width instead of colliding below 1440px.
     --br  starts at 65%, so a flat 35vw puts its right edge exactly on the
           screen edge at any width up to 1200px — and its corner marks, which
           translate 50% outward, would be cut in half there.
   Above ~1590px both caps stop binding and the 420px ceiling takes over, so the
   wide layout is unchanged. The fish's swim animation only ever moves it right
   and up (+18/-18), so it never eats into the clearance. */
.about__box--tr { left: 43%; top: 7%; width: min(420px, calc(26vw + 38px)); aspect-ratio: 126 / 66; }
.about__box--bl { left: 12%; bottom: 7%; width: min(34vw, 405px); aspect-ratio: 108 / 69; }
.about__box--br { left: 65%; bottom: 9%; width: min(420px, calc(31vw - 12px)); aspect-ratio: 126 / 63; }
/* the phone frame's fourth card: the desktop collage has no empty corner to
   fill, so it only exists inside the media query below */
.about__box--tr2 { display: none; }

/* fills the framed box. ScratchCard's own root already does this for itself;
   this is what gives the plain <div> that replaces it on mobile the same box
   (a parent's scoped styles do reach a child component's root element). */
.about__card {
  position: absolute;
  inset: 0;
}

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
  font-size: clamp(11px, 1.3vw, 20px);
  line-height: 1.35;
}
.about__box-caret { opacity: var(--blink); }

/* mobile: its own frame in Figma, not the desktop collage rescaled. About is
   rendered inside App.vue's reveal, which is a position: fixed, clip-path'd
   window — so this section can never scroll. Anything that doesn't fit in one
   screen isn't just ugly, it is permanently unreachable. Everything therefore
   stays absolutely positioned in percentages of the section's own height
   (= one viewport): cat and card up top, the title across the middle, fish
   and second card below it, 777 in the bottom corner. */
@media (max-width: 760px) {
  /* ONE unit for the whole composition: --u. Every size and every vertical
     offset below is a multiple of it, so the frame can only ever move as a
     single rigid picture — nothing can drift into anything else, because
     nothing moves relative to anything else.

     --u: min(1vw, 0.5155cqh)

     The two terms are the two things that can go wrong, and the `min` picks
     whichever one is binding:

     · 1vw is the design size. It is what every number here was measured at, and
       width is the one dimension Safari's URL bar never touches — so on a real
       phone this term wins and the elements stay exactly as big as the frame
       draws them, with the bar out or in. Sliding the bar changes nothing at
       all now, which is the point: sizing off the height (as this block did a
       moment ago) shrank everything ~10% every time the bar appeared.

     · 0.515cqh is the escape hatch. The stack below runs from 0 to 192.6u —
       the 777 is the last thing on it, and its real line box measures 14.6u,
       not the 13.7u of its font-size — so it fits the section when
       u = 100cqh / 194 = 0.515cqh, a hair of slack included. On a
       screen too short for the design — a small phone, a landscape one — that
       term goes under 1vw and the entire picture scales down together rather
       than letting the bottom overlap or fall off. cqh is the section's real
       current height (container-type: size below, on a box that is fixed and
       inset: 0), so it needs no svh/lvh/dvh guess.

     The margin is deliberate: 0.515cqh only drops under 1vw below ~1.94x the
     width in visible height, and a phone with its bar showing is still taller
     than that or within a fraction of a percent of it. So the whole range the
     bar moves through is on the vw side — full size, frozen.

     Horizontal offsets stay in plain %, of width, for the same reason: width
     holds still, and holding it still is what keeps the columns lined up. */
  .about {
    container-type: size;
    /* The fondo moves up to .reveal__about (see App.vue). This element is the
       one that rides up as Projects arrives, and a background travels with the
       element it is painted on — so leaving the lattice here meant the grid
       scrolled away under the words while the fixed one behind it stood still,
       which is the drift that made the handover feel loose. The parent is
       `position: fixed; inset: 0` and never transformed, so painting it there
       gives the same picture with the background nailed to the screen: the
       composition leaves, the paper it is drawn on does not. */
    background-color: transparent;
    background-image: none;
  }

  /* The four dots belong to the screen, not to this section. They are children
     of .about, so the ride-up carries them off with everything else — and since
     Projects no longer draws its own pair, the corners would simply go empty
     for the rest of the page. --leave is the very translation App.vue applies
     to .about, published as a custom property and therefore inherited down to
     here, so translating by +--leave against the parent's ---leave cancels it
     exactly, whatever the scroll offset. The marks are then nailed to the four
     corners of the viewport from the moment the reveal opens until the end of
     the CTA: one set of dots for every view, which is the point. */
  .about__corner {
    transform: translateY(var(--leave, 0px));
  }
  /* on the children, not on .about itself: cqh resolves against an *ancestor*
     container, so the declaration has to sit inside the container it queries.
     Custom properties inherit, so the grandchildren (card texts, badge text)
     pick it up from here. */
  .about > * {
    --u: min(1vw, 0.515cqh);
  }

  .about__dvd { font-size: clamp(18px, calc(5.5 * var(--u)), 30px); }

  /* the cut-outs are placed by where their INK lands, not by their file's box:
     both carry a wide transparent margin. cat.png starts drawing 4.76% in and
     12.38% down, covering 90.5% x 75.2% of its canvas; fish.png, 13.1% and
     23.8% in, covering 77.4% x 52.4%. Each offset here is the position the
     frame asks for minus that margin — otherwise the invisible bounding box
     is what lines up and the animal itself sits low and to the right. The
     widths are likewise inflated so the ink, not the file, measures right. */
  .about__cat {
    left: 50.3%; /* puts the ink's left edge at 52.6% */
    top: calc(33 * var(--u));
    width: calc(50 * var(--u)); /* ink reads 43u wide, flush with the frame */
  }
  .about__fish {
    left: -4%; /* ink's left edge at 4.6%, level with the corner marks */
    top: calc(98 * var(--u));
    width: calc(54.3 * var(--u)); /* ink reads 42u wide */
  }

  .about__title {
    top: calc(100 * var(--u));
    font-size: clamp(30px, calc(13 * var(--u)), 64px);
  }
  /* 'Faster One' measures 2.11em across for "777" (measured, not guessed — its
     advance is nothing like DotGothic's flat 0.5em), and the frame wants that
     block 29u wide, so 29 / 2.11 is the size that reproduces it */
  .about__777 {
    left: 60%;
    top: calc(165 * var(--u));
    font-size: clamp(32px, calc(13.7 * var(--u)), 62px);
  }

  /* the badge moves to the top-left corner, in the band above the first card
     (which starts at 20%) and to the left of the cat (which starts at 50.3vw),
     so it lands in the one empty region of this frame. Aligned at 4vw with the
     fish's ink and the corner marks below it. */
  .about__badge {
    left: 11%;
    top: calc(6.37 * var(--u));
    width: calc(40 * var(--u));
  }
  /* the star's readable area is only its inner radius, 0.73 of the box = 74px
     on a 375pt screen. At 3.8vw the longer line, "ERES???", measures 58 of
     those 74 even after the -14deg tilt, so it fills the star without touching
     the points. The lower floor is there for a 320pt screen, where the desktop
     14px minimum would push the word out past them. */
  .about__badge-text {
    font-size: clamp(1px, calc(6 * var(--u)), 18px);
  }

  /* two cards, square like the frame draws them, opposite corners of the
     title. The dropped one is the QUIEN ERES??? answer — its text lives in the
     desktop scratch cards, and this frame only has room for two. */
  .about__box {
    /* the old min(..., 33svh) cap is gone: --u already carries the height
       constraint for the whole composition at once, so a card can no longer be
       the one thing that outgrows a landscape screen */
    width: calc(40 * var(--u));
    height: auto;
    aspect-ratio: 1 / 1;
    bottom: auto;
  }
  .about__box--tr { left: 8%; top: calc(48.8 * var(--u)); }
  .about__box--br { left: 55%; top: calc(114 * var(--u)); }
  .about__box--bl { left: 10%; top: calc(145 * var(--u)); }

  /* the fourth card, in the band above the cat. That band is 32u deep — the
     cat's file box starts at 24.6u but its ink only at 31.95u, the rest being
     transparent margin — so a 24u-tall card at 4u clears the drawing by 4u, and
     sits above it anyway (z-index 1 against the cat's 0). Wide and short rather
     than square: it is an aside, and it has a whole corner to run across but
     very little height before the cat. */
  .about__box--tr2 {
    display: block;
    left: 60%;
    top: calc(8 * var(--u));
    width: calc(36 * var(--u));
    height: calc(24 * var(--u));
    aspect-ratio: auto;
  }

  /* percentage padding resolves against width on every side, so the desktop
     10% would eat too much of these smaller cards */
  .about__box-text {
    /* the padding scales with the card too — fixed px would eat a growing
       share of it as the card shrinks, and the text would clip before the
       card ever looked small */
    padding: calc(2.1 * var(--u)) calc(2.7 * var(--u));
    /* the floor has to stay under the card's own scaling or it stops shrinking
       while the card keeps going: at a 10px floor the longest bio overflowed a
       568-tall screen by 11px */
    font-size: clamp(8px, calc(3.2 * var(--u)), 16px);
    line-height: 1.35;
  }
}
</style>
