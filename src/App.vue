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

// One unit of choreography, in pixels. It has to be the very same number the
// layout reserves room with, and it must not move while a gesture is in
// flight. window.innerHeight is neither on a phone: iOS collapses the URL bar
// mid-scroll and innerHeight grows from the small viewport to the large one
// underneath us, while the CSS below had reserved its room in svh. The reveal's
// 100% then landed at a different scroll offset than the two spacers implied,
// and Projects — whose top edge sits exactly two units down — came into view
// with the window still ~80% open. Measuring the spacer reads the layout's own
// answer instead of guessing, and lvh (see the stylesheet) is the one viewport
// unit iOS does NOT redefine as the bar collapses, so it holds still.
const unit = ref(window.innerHeight || 1)

// What a `position: fixed; inset: 0` box actually measures right now, and where
// Projects starts in the document. The About rides up on a fixed box; Projects
// arrives on document scroll. Joining them means knowing both numbers, and only
// one of them is `unit`.
const fixedH = ref(window.innerHeight || 1)
const projectsTop = ref(0)

// Vertical offset that puts Projects' grid back in phase with the About's.
//
// Four elements on this page paint the same lattice, and a CSS background is
// anchored to its own element's top-left. The hero, the backdrop and the About
// are all `position: fixed; inset: 0`, so all three anchor at viewport y = 0 and
// their lines coincide — which is why the reveal window opens without seaming.
// .projects-wrap anchors at its own top instead, and its top reaches the screen
// exactly `fixedH` below the About's. 844 % 15 = 4, so Projects used to arrive
// with its grid 4px off, and that broken line rode up the screen for the whole
// length of the handover. Since the two sections now travel together, cancelling
// the offset once is enough to make the lattice continuous across the seam
// forever — no per-frame work, and nothing to recompute while scrolling.
const gridPhase = ref(0)

function measureUnit() {
  const spacer = document.querySelector<HTMLElement>('.reveal-spacer')
  unit.value = spacer?.offsetHeight || window.innerHeight || 1
  // .viewport-probe carries exactly the CSS .reveal does, so its height is the
  // reveal's height by construction — whatever iOS currently means by a fixed
  // box, with no svh/dvh/innerHeight guessing, and readable before .reveal is
  // even mounted
  const probe = document.querySelector<HTMLElement>('.viewport-probe')
  fixedH.value = probe?.offsetHeight || window.innerHeight || 1
  const wrap = document.querySelector<HTMLElement>('.projects-wrap')
  if (wrap) {
    projectsTop.value = wrap.getBoundingClientRect().top + window.scrollY
    // Prefer the cell the element is actually painted with, so this can never
    // drift from the stylesheet — but fall back to the clamp's own arithmetic.
    // Read during onMounted the computed value can still come back with its
    // viewport units unresolved, and parseFloat of that is NaN; taking the
    // fallback as 0 left the phase at 0 and the seam 4px out on first load.
    const painted = parseFloat(getComputedStyle(wrap).backgroundSize)
    const cell = Number.isFinite(painted) && painted > 0
      ? painted
      : Math.min(28, Math.max(15, 0.02 * window.innerWidth))
    gridPhase.value = -(fixedH.value % cell)
  }
}

// the reveal is driven purely by scroll position, so a single hard flick on a
// phone can carry the page from closed to well past fully-open in one gesture:
// the window snaps open and is immediately ridden off the top by Projects, and
// the section is never actually seen. Catch the first moment it reaches 100%,
// pin it there, and hold the page still long enough for the flick's momentum
// to die out — going on to Projects then takes a fresh swipe. One-shot: once
// it has been paid, scrolling back up and down again is unobstructed.
const BRAKE_MS = 500
let braking = false
let braked = false

function brake(u: number) {
  braking = true
  braked = true
  setScrollLock(true)
  window.setTimeout(() => {
    braking = false
    setScrollLock(false)
    // locking the document can clamp its scroll offset; resume from exactly
    // where the reveal finished rather than from wherever it was left
    if (Math.abs(window.scrollY - u) > 1) window.scrollTo(0, u)
  }, BRAKE_MS)
}

// setScrollLock's overflow: hidden genuinely freezes a desktop scroller, but on
// iOS the fling that follows a swipe is run by the compositor and sails right
// through it. Cancelling touchmove is what actually stops a finger there.
function onTouchMove(e: TouchEvent) {
  if (braking && e.cancelable) e.preventDefault()
}

function onScroll() {
  const u = unit.value
  if (!braked && heroDone.value && window.scrollY >= u) brake(u)
  if (braking) {
    // and this is what kills momentum already in flight: every scroll event
    // the fling emits, put the document straight back where it belongs
    if (window.scrollY !== u) window.scrollTo(0, u)
    // hold the composition fully open whatever the locked document reports —
    // otherwise a clamped scroll offset would read as the window closing again
    scrollY.value = u
    progress.value = 1
    return
  }
  scrollY.value = window.scrollY
  progress.value = Math.min(1, Math.max(0, window.scrollY / u))
}
const aboutActive = computed(() => progress.value > 0.3)

// Once fully open, the About rides up with the scroll so Projects replaces it —
// and the two must leave and arrive as one piece.
//
// The scroll offset at which Projects' top edge touches the bottom of the
// visible viewport is (projectsTop - fixedH), and from there on the About has
// to rise by exactly what the page scrolls. Its bottom edge is then always at
// (fixedH - leaveOffset) and Projects' top edge at (projectsTop - scrollY), and
// those are the same number by construction: joined, on any device, at any
// point in the gesture.
//
// This used to start the ride at `unit`, which is the same thing only when the
// spacer's 100lvh equals the visible viewport. On a phone with the URL bar
// showing it does not: `unit` is the ~80px taller large viewport, so the About
// lifted off 80px before Projects had arrived and a white band opened between
// them. On desktop lvh is the visible viewport, this resolves back to `unit`
// exactly, and nothing about the wide layout changes.
const joinScroll = computed(() => Math.max(unit.value, projectsTop.value - fixedH.value))
const leaveOffset = computed(() => Math.max(0, scrollY.value - joinScroll.value))

// keep the page unscrollable until the hero has finished typing, so the reveal
// can only begin (from zero) once the text is loaded — never mid-way
function setScrollLock(lock: boolean) {
  document.documentElement.style.overflow = lock ? 'hidden' : ''
}
// Both of these also re-measure, and it is worth being clear why: onMounted
// takes a single sample, and if the viewport happens to be mid-flight at that
// instant — a phone still settling, a desktop window being dragged — every
// number stays wrong until something moves again. These two fire seconds later,
// off timers rather than off the rendering pipeline, at moments when the layout
// is certainly settled and nothing is being scrolled. Free insurance behind the
// ResizeObserver, and independent of it.
watch(introDone, (v) => {
  if (v && !heroDone.value) setScrollLock(true)
  measureUnit()
})
watch(heroDone, (v) => {
  if (v) setScrollLock(false)
  measureUnit()
})

// Width-only gate on the window resize listener.
//
// The comment on that listener used to read "the URL bar collapsing does not
// fire this, so re-measuring here never moves mid-gesture". That is true of iOS
// Safari and false of Chrome on Android, which is the one behavioural split
// between the two that this component actually depends on: on Android the bar
// sliding away DOES fire resize, so measureUnit() ran in the middle of a swipe.
// It re-derives projectsTop as getBoundingClientRect().top + window.scrollY, and
// while the bar is animating the visual and layout viewports are a few frames
// out of step — the two halves of that sum then refer to different frames and
// the total is wrong, which corrupts joinScroll and with it the whole handover.
//
// Nothing is lost by skipping those: a height-only change is by definition a
// change to a `position: fixed; inset: 0` box, so the ResizeObserver on
// .viewport-probe catches it, and visualViewport's own resize catches it again.
// This listener is left doing what only it can do — rotations and real window
// resizes, both of which change the width. Strictly less work on every
// platform, since measureUnit() forces a synchronous layout each time it runs.
let lastWidth = window.innerWidth
function onWindowResize() {
  if (window.innerWidth === lastWidth) return
  lastWidth = window.innerWidth
  measureUnit()
}

let probeObserver: ResizeObserver | null = null

onMounted(() => {
  // always start the flow from the top so the reveal plays from zero
  // (browsers otherwise restore the previous scroll position on reload)
  if ('scrollRestoration' in history) history.scrollRestoration = 'manual'
  window.scrollTo(0, 0)
  progress.value = 0
  measureUnit()
  const probeEl = document.querySelector<HTMLElement>('.viewport-probe')
  window.addEventListener('scroll', onScroll, { passive: true })
  // passive: false or preventDefault() is ignored — that is the whole point
  window.addEventListener('touchmove', onTouchMove, { passive: false })
  // rotations and real window resizes only — see onWindowResize above for why
  // the height-only ones are deliberately dropped here
  window.addEventListener('resize', onWindowResize)
  window.addEventListener('orientationchange', measureUnit)
  // the URL bar collapsing resizes a fixed box without firing window resize;
  // this is the event that does fire, and re-measuring there is what keeps the
  // About and Projects glued while the bar comes and goes
  window.visualViewport?.addEventListener('resize', measureUnit)

  // ...and this is what catches the ones that fire no event at all. Every
  // number this component works with — unit, fixedH, projectsTop, the grid
  // phase — describes the viewport as it was the last time an event happened to
  // arrive, and a viewport can finish changing without sending one: measured on
  // load, the box went 844 -> 675 -> 844 and only the first two were reported,
  // so the layout was left being driven by a 675px viewport that no longer
  // existed. Watching the probe's box instead of listening for announcements of
  // it means the measurement cannot go stale — whatever moves it, however it
  // moves, this fires. Which matters most on a phone, where the URL bar sliding
  // away is precisely a burst of intermediate heights.
  if (probeEl && 'ResizeObserver' in window) {
    probeObserver = new ResizeObserver(measureUnit)
    probeObserver.observe(probeEl)
  }
})
onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('touchmove', onTouchMove)
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('orientationchange', measureUnit)
  window.visualViewport?.removeEventListener('resize', measureUnit)
  probeObserver?.disconnect()
})
</script>

<template>
  <IntroSequence v-if="!introDone" @done="introDone = true" />

  <div class="page">
    <!-- measured, never seen: see .viewport-probe in the stylesheet -->
    <div class="viewport-probe" aria-hidden="true" />

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
      :style="{ '--p': progress, '--leave': leaveOffset + 'px' }"
    >
      <div class="reveal__about">
        <About :active="aboutActive" />
      </div>
      <div class="reveal__frame" />
    </div>

    <!-- scroll room for the box to finish opening before Projects arrives -->
    <div class="reveal-spacer" />

    <!-- Projects scrolls up over the opened About -->
    <div class="projects-wrap" :style="{ '--grid-phase': gridPhase + 'px' }">
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
/* One viewport of scroll for the reveal to open before Projects comes up.
   lvh, not svh: this box is pure scroll room, never seen, and it has to be at
   least as tall as the largest the visible viewport can get. Reserved in svh it
   was 80px short of that on a phone, so Projects' top edge slid into view while
   the reveal still had 20% to go. lvh also holds still while iOS grows and
   shrinks the visible area, which is what keeps the choreography from sliding
   around as the URL bar comes and goes. On desktop lvh == svh == vh, so
   nothing about the wide layout changes. measureUnit() in the script reads this
   element's height so both sides agree on one number. */
.reveal-spacer {
  height: 100lvh;
}
/* An invisible stand-in for .reveal, carrying the same position: fixed; inset:0
   so that its height IS the reveal's height, whatever the browser currently
   takes a fixed box to be. measureUnit() reads it. It has to be always present
   (the .reveal itself is behind a v-if and does not exist when the number is
   first needed) and it has to take part in layout, which is why this is
   visibility: hidden and not display: none. */
.viewport-probe {
  position: fixed;
  inset: 0;
  z-index: -1;
  visibility: hidden;
  pointer-events: none;
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
  /* see gridPhase in the script: lines up this lattice with the About's, so the
     handover has no step in it. Defaults to 0 for the first paint, before
     measureUnit() has run. */
  background-position: 0 var(--grid-phase, 0px);
}

/* --leave (the pixels the composition has ridden up by, published from the
   script) is set on this element rather than the transform itself, because two
   different elements need to read it: on desktop the whole layer moves, on
   mobile only the contents do while the background stays nailed to the screen.
   A custom property inherits, so both can be expressed in CSS from the one
   number, and the mobile override is a rule rather than a second binding. */
.reveal {
  position: fixed;
  inset: 0;
  z-index: 5;
  pointer-events: none;
  animation: reveal-in 0.25s ease-out;
  transform: translateY(calc(-1 * var(--leave, 0px)));
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
  /* Percentages, not viewport units. This border traces the window that
     .reveal__about's clip-path cuts, and that clip is a percentage of the
     fixed box both elements share — so expressing the border the same way
     makes the two identical by construction, on any device and whatever iOS
     decides a fixed box measures while its URL bar comes and goes. Written in
     vw/svh/lvh it can only ever be a guess at that number, and every pixel the
     guess is off shows up as a gap between the orange line and the edge of
     what has been revealed. Same 1% -> 100% ramp as the clip: at --p = 0 a
     1% sliver, at 1 the whole box. */
  width: calc(1% + 99% * var(--p));
  height: calc(1% + 99% * var(--p));
  border: 2px solid #ff3d00;
  box-sizing: border-box;
  pointer-events: none;
}
.reveal.is-open .reveal__frame {
  opacity: 0;
}

/* ---------------------------------------------------------------------
   mobile overrides — MUST stay at the end of this stylesheet. A media
   query adds no specificity, so these rules only beat the base ones above
   by coming later in source order. Moving this block up silently reverts
   every override in it.
   --------------------------------------------------------------------- */

/* some decorative elements deliberately peek past their section's edge
   (e.g. Projects' corner star) — harmless on desktop, but on a narrow
   viewport that can make the page pannable sideways. `overflow-x: hidden`
   alone doesn't stop this on iOS Safari (it still lets you swipe/rubber-band
   the page horizontally); `touch-action: pan-y` is what actually disables
   horizontal touch panning there — and, since `pan-y` excludes `pinch-zoom`,
   it also stops the accidental zoom-out that leaves the layout stranded in a
   corner of a white screen. overflow-y is set explicitly alongside overflow-x
   to avoid the CSS quirk where leaving it unset silently switches it to auto
   (which would break position: sticky elsewhere on the page). */
@media (max-width: 760px) {
  html,
  body {
    overflow-x: hidden;
    overflow-y: auto;
    touch-action: pan-y;
  }

  /* pin the hero to the viewport on mobile. position: sticky still resolves
     its offset against scroll every frame, which on a phone reads as the
     section drifting; position: fixed decouples it from scroll entirely, so
     it cannot shift sideways, up or down while it is showing.
     height: auto is deliberate — with top/bottom both at 0 the box then
     measures the *currently visible* viewport, so the hero keeps filling the
     screen exactly as the browser's URL bar collapses, with no svh/dvh
     guessing and no white strip appearing under it mid-scroll. */
  .page__hero {
    position: fixed;
    inset: 0;
    height: auto;
  }

  /* a fixed hero contributes no height to the document, which would eat the
     first viewport of scroll the reveal choreography runs on. This gives
     that exact amount back, so the scroll geometry stays identical to
     desktop: first screen of scroll opens the About window over the pinned
     hero, second screen (.reveal-spacer) rides it up as Projects arrives. */
  .page {
    padding-top: 100lvh;
  }

  /* -------------------------------------------------------------------
     One background for the whole phone.

     The page used to paint the same lattice on four separate elements, and
     two of them moved: .reveal rode up on a transform, taking the About's
     grid with it, while the fixed backdrop behind it stayed put — and
     .projects-wrap scrolled its own copy up on top of that. Three lattices
     at three different speeds. Individually each one is defensible; together
     they read as the whole page coming loose from its own background, which
     is exactly what it felt like.

     So on mobile there is one lattice and it is nailed to the screen.
     .reveal__about is `position: fixed; inset: 0` and is no longer the
     element that moves, so it is the right place to paint it: the About's
     own background comes off (About.vue), the composition alone rides up,
     and Projects arrives over the same still paper instead of bringing more
     of its own. Nothing can drift against anything now, because there is
     nothing to drift against — which is also why gridPhase stops mattering
     here, though it still earns its keep on desktop.
     ------------------------------------------------------------------- */
  .reveal {
    /* the layer holds still; only what is drawn on it leaves */
    transform: none;
  }
  .reveal__about {
    background-color: #ffffff;
    background-image:
      linear-gradient(to right, rgba(0, 0, 0, 0.13) 1px, transparent 1px),
      linear-gradient(to bottom, rgba(0, 0, 0, 0.13) 1px, transparent 1px);
    background-size: clamp(15px, 2vw, 28px) clamp(15px, 2vw, 28px);
  }
  /* the About component's root: it carries the ride-up now, so the background
     it sits on can stay behind. Same motion as before, one level down. */
  .reveal__about > * {
    transform: translateY(calc(-1 * var(--leave, 0px)));
  }
  /* transparent, so what shows through is the fixed lattice above rather than
     a second one travelling with the scroll. Nothing is lost by it: by the
     time Projects covers any part of the screen the About's composition has
     already ridden past that line, so there was never anything here to hide. */
  .projects-wrap {
    background-color: transparent;
    background-image: none;
  }
}
</style>
