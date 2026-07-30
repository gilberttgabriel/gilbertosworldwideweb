<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import calmEyes from '../assets/eyes/calm.png'
import doubtEyes from '../assets/eyes/doubt.png'
import seriousEyes from '../assets/eyes/serious.png'

const emit = defineEmits<{ done: [] }>()

type Stage = 'idle' | 'loading' | 'leaving'
const stage = ref<Stage>('idle')

const currentArt = ref(seriousEyes)

const loadingPercent = ref(0)

const stageEl = ref<HTMLElement | null>(null)
const dotsExpanded = ref(false)
const showDots = ref(false)

async function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms))
}

const LOADING_MS = 3000
// keeps the eyes changing expression for the whole loading bar duration,
// instead of sitting frozen on one static image while the user waits
const loadingArtCycle = [seriousEyes, doubtEyes, calmEyes, doubtEyes]
function runLoadingEyes() {
  let i = 0
  const id = setInterval(() => {
    i = (i + 1) % loadingArtCycle.length
    currentArt.value = loadingArtCycle[i] ?? seriousEyes
  }, LOADING_MS / (loadingArtCycle.length * 2))
  return () => clearInterval(id)
}

async function runLoadingBar() {
  const start = performance.now()
  return new Promise<void>((resolve) => {
    // a timer, not requestAnimationFrame: rAF stops dead while the tab isn't
    // visible, so glancing away or letting the screen lock during these three
    // seconds stranded the bar at 0% and the intro never handed off to the
    // hero — the page just sat on the eyes forever. Timers keep firing when
    // hidden (throttled, but firing), and progress is read off the clock
    // rather than counted per tick, so it still lands exactly on time however
    // irregular those ticks turn out to be.
    const id = setInterval(() => {
      const t = Math.min(1, (performance.now() - start) / LOADING_MS)
      loadingPercent.value = Math.round(t * 100)
      if (t >= 1) {
        clearInterval(id)
        resolve()
      }
    }, 50)
  })
}

async function runSequence() {
  await sleep(500)

  // loading bar takes over from the old "quien eres???" question — the
  // eyes keep cycling expressions the whole time so it doesn't feel static
  stage.value = 'loading'
  const stopEyes = runLoadingEyes()
  await runLoadingBar()
  stopEyes()

  currentArt.value = calmEyes

  // 4 dots appear solidly (centered) once the loading bar has finished
  showDots.value = true
  await sleep(200)

  // eyes fade away, dots stay put
  stage.value = 'leaving'
  await sleep(300)

  // then the dots expand out to the corners
  dotsExpanded.value = true
  await sleep(750)
  // dots have landed on the corners — hand off to the Hero's corner marks
  // (which sit at the exact same spots) so the swap is seamless
  emit('done')
}

onMounted(() => {
  document.body.style.overflow = 'hidden'
  runSequence()
})
onUnmounted(() => {
  document.body.style.overflow = ''
})
</script>

<template>
  <div ref="stageEl" class="intro" :class="{ 'is-leaving': stage === 'leaving' }">
    <div class="intro__stack">
      <div class="intro__art">
        <img :src="currentArt" alt="" class="intro__img" draggable="false" />
      </div>

      <div class="intro__caption-slot" aria-live="polite">
        <div v-if="stage === 'loading'" class="intro__loading">
          <div class="intro__loading-track">
            <div class="intro__loading-fill" :style="{ width: loadingPercent + '%' }" />
          </div>
        </div>
      </div>
    </div>

    <div
      v-if="showDots"
      class="intro__transition-dots"
      :class="{ 'is-expanded': dotsExpanded }"
    >
      <span class="transition-dot transition-dot--tl" />
      <span class="transition-dot transition-dot--tr" />
      <span class="transition-dot transition-dot--br" />
      <span class="transition-dot transition-dot--bl" />
    </div>
  </div>
</template>

<style scoped>
.intro {
  position: fixed;
  inset: 0;
  background: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(24px, 8vh, 96px) clamp(16px, 6vw, 64px);
  box-sizing: border-box;
  z-index: 1000;
  overflow: hidden;
  transition: background-color 0.4s ease 0.15s;
}
.intro.is-leaving {
  background-color: transparent;
}

.intro__stack {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: min(66vw, calc(52vh * 1112 / 525));
  max-width: 94vw;
  transform: translateY(-6vh);
  transition: opacity 0.3s ease;
}
.is-leaving .intro__stack {
  opacity: 0;
}

.intro__art {
  position: relative;
  width: 100%;
  aspect-ratio: 1112 / 525;
}

.intro__img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  user-select: none;
  pointer-events: none;
}

.intro__caption-slot {
  flex-shrink: 0;
  height: 76px;
  margin-top: 4px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  gap: 10px;
  width: max-content;
}

.intro__loading {
  width: clamp(160px, 26vw, 260px);
}
.intro__loading-track {
  width: 100%;
  height: 4px;
  background: #e5e2da;
  border-radius: 2px;
  overflow: hidden;
}
.intro__loading-fill {
  height: 100%;
  background: #ff3d00;
  transition: width 0.1s linear;
}

.intro__transition-dots {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 2;
  /* the whole group fades in solid, no per-dot scaling or motion */
  animation: dots-appear 0.18s ease-out;
}
@keyframes dots-appear {
  from { opacity: 0; }
  to { opacity: 1; }
}

.transition-dot {
  position: fixed;
  width: 5px;
  height: 5px;
  background: #ff3d00;
  transform: translate(-50%, -50%);
  transition:
    left 0.7s cubic-bezier(0.5, 0, 0.2, 1),
    top 0.7s cubic-bezier(0.5, 0, 0.2, 1);
}

/* start: a tight 2x2 cluster centered near eye level */
.transition-dot--tl { left: calc(50% - 4.5px); top: calc(46% - 4.5px); }
.transition-dot--tr { left: calc(50% + 4.5px); top: calc(46% - 4.5px); }
.transition-dot--br { left: calc(50% + 4.5px); top: calc(46% + 4.5px); }
.transition-dot--bl { left: calc(50% - 4.5px); top: calc(46% + 4.5px); }

/* expanded: corners match Hero's corner marks exactly */
.is-expanded .transition-dot--tl { left: clamp(12px, 2.5vw, 32px); top: clamp(12px, 2.5vw, 32px); }
.is-expanded .transition-dot--tr { left: calc(100vw - clamp(12px, 2.5vw, 32px)); top: clamp(12px, 2.5vw, 32px); }
.is-expanded .transition-dot--br { left: calc(100vw - clamp(12px, 2.5vw, 32px)); top: calc(100vh - clamp(12px, 2.5vw, 32px)); }
.is-expanded .transition-dot--bl { left: clamp(12px, 2.5vw, 32px); top: calc(100vh - clamp(12px, 2.5vw, 32px)); }

@media (max-width: 760px) {
  /* Centering the welcome, measured rather than guessed.
     serious.png's ink leaves 8.99% transparent on the left and 7.73% on the
     right: the drawing does sit 1.26% of its width off-centre in its own
     canvas, so half of that — 0.63%, 1.6px on a phone — is the entire
     horizontal correction it ever needed. The translateX(-5vw) that used to
     live here was eight times that, which is why the eyes read as shoved to
     the left. It moves with the stack now rather than being applied to the art
     and the caption slot separately, so the loading bar cannot drift out of
     line with the eyes above it.

     Vertically the ink runs 42.10% -> 84.95% down the canvas; the top 42% is
     empty. So flex-centring the .intro__stack centres a box that is mostly
     transparent air, and the visible composition floats high — the -6vh on
     top of that pushed it higher still. What should be centred is what can be
     seen: the eyes plus the loading bar.

       art height  = --w / (1112/525) = 0.4721 * --w
       group       = ink top (0.4210 * art) -> bar bottom (art + 8px)
       its centre sits 0.2105 * art - 36px below the stack's own centre

     so translating back by that amount lands the composition on the middle of
     the screen. 0.2105 * 0.4721 = 0.09937 of --w. --w restates the base
     width so the correction holds on a short screen too, where the 52vh term
     wins and the art is smaller than 66vw would make it. */
  .intro__stack {
    --w: min(66vw, calc(52vh * 1112 / 525), 94vw);
    transform: translate(calc(-0.0063 * var(--w)), calc(36px - 0.09937 * var(--w)));
  }

  /* the cluster starts where the eyes are, so it flows to the corners in one
     motion instead of jumping first. With the stack centred the ink's middle
     lands a touch above the screen's, not at the old 46%. */
  .transition-dot--br,
  .transition-dot--bl {
    top: calc(48% + 9px);
  }
  .transition-dot--tl,
  .transition-dot--tr {
    top: calc(48% - 9px);
  }

  /* the dots are position: fixed, so they land against the edge of what's
     actually on screen — but 100vh on a phone measures the taller layout
     viewport that continues behind the browser's URL bar, so the bottom two
     would fly off past the visible edge and miss the Hero's corner marks
     they're supposed to hand off to. dvh is the edge the user can see. */
  .is-expanded .transition-dot--br,
  .is-expanded .transition-dot--bl {
    top: calc(100dvh - clamp(12px, 2.5vw, 32px));
  }
}
</style>
