<script setup lang="ts">
import { ref } from 'vue'
import folderImg from '../assets/folder.png'
import minecraftImg from '../assets/minecraft.png'
import amadtImg from '../assets/amadt-screenshot.png'
import avranPortfolioImg from '../assets/avransportfolio.png'
import gilbertosImg from '../assets/gilbertos.png'

// Blue burst of the mobile frame, traced from assets/BLUE STAR.webp. That file
// is lossy VP8 with no alpha channel — a white rectangle with a star painted on
// it — so dropping it in as an <img> would punch a white box through the grid
// background and through the words it is meant to sit on top of. These are its
// own 22 vertices (11 rays), read off the bitmap's outer contour and simplified
// until they cover 98.5% of the original's area, normalised into a 1000x1000
// box. Its hub — the point the rays radiate from, which is what has to land on
// a given spot, not the bbox centre — sits at 56.4% / 47.8% of that box.

const burstPoints =
  '774.7,0 729,277.6 970.2,226.2 762,391.2 1000,426.6 788.5,537 938.4,677.4 ' +
  '718.4,651.7 854.4,933.5 578.1,728.8 348.6,1000 453.8,663.5 120.1,837.1 ' +
  '351.8,587.4 0,531.6 319.9,464.1 135,320.5 362.4,334.4 190.2,69.7 475,266.9 ' +
  '508,81.5 562.2,250.8'

// yellow burst pinned in the corner: 14-point star, inner ratio 0.63 (Figma)
const starPoints = (() => {
  const cx = 100, cy = 100, outer = 100, inner = 63, rays = 14
  const pts: string[] = []
  for (let i = 0; i < rays * 2; i++) {
    const r = i % 2 === 0 ? outer : inner
    const a = (Math.PI / rays) * i - Math.PI / 2
    pts.push(`${(cx + r * Math.cos(a)).toFixed(2)},${(cy + r * Math.sin(a)).toFixed(2)}`)
  }
  return pts.join(' ')
})()

const projects = [
  {
    title: 'AMADT',
    image: amadtImg,
    description:
      'amadt.com es un proyecto web para el álbum AMADT de Male y Alejori. Los muchachos querían una experiencia parecida a estar en un museo, en donde cada canción actuara como un cuadro. Dentro de estos cuadros está toda la información relevante al tema y los links que redirigen a las plataformas musicales. Cuenta con frontend y backend; tanto el diseño como el desarrollo de esta página fueron hechos por mí.',
  },
  {
    title: 'Avran\'s Portfolio',
    image: avranPortfolioImg,
    imagePosition: 'center center',
    description:
      'Es un portafolio creativo para Abraham Ríos, director creativo y estilista. Está en proceso, sin embargo pronto estará listo para despliegue. Abraham me envió un tablero de Pinterest y referencias personales, y a partir de ahí diseñé y desarrollé su portafolio. Si quieres uno, hit me up :)',
  },
  {
    title: 'gilbertosworldwideweb',
    image: gilbertosImg,
    description: 'Bueno, es esta misma página. Gracias por leer hasta acá, el próximo proyecto puede ser el tuyo.',
  },
]

const activeProject = ref<(typeof projects)[number] | null>(null)
function openProject(p: (typeof projects)[number]) {
  activeProject.value = p
}
function closeProject() {
  activeProject.value = null
}
</script>

<template>
  <section id="proyectos" class="projects">
    <!-- full-screen cover: PROYECTOS is its own view, scrolled past like a
         title card, before any project cards appear -->
    <div class="projects__cover">
      <svg class="projects__star" viewBox="0 0 200 200" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
        <polygon :points="starPoints" fill="#F8E10E" />
      </svg>

      <!-- mobile stand-in for the yellow star above (see the media query) -->
      <svg class="projects__burst" viewBox="0 0 1000 1000" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
        <polygon :points="burstPoints" fill="#2415ff" />
      </svg>

      <span class="projects__word projects__word--pro">PRO</span>
      <span class="projects__word projects__word--yec">YEC</span>
      <span class="projects__word projects__word--tos">TOS</span>

      <img :src="folderImg" class="projects__folder" alt="" draggable="false" />
      <img :src="minecraftImg" class="projects__mc" alt="" draggable="false" />

      <!-- the frame's corner marks. They exist site-wide on the Hero, but on
           mobile Projects scrolls over the fixed Hero with an opaque background
           and takes them off screen with it, so this view carries its own. -->
      <span class="projects__corner projects__corner--tl" />
      <span class="projects__corner projects__corner--tr" />
      <span class="projects__corner projects__corner--br" />
      <span class="projects__corner projects__corner--bl" />
    </div>

    <!-- stacked list of project cards, following the cover in normal scroll -->
    <div class="projects__list">
      <article
        v-for="p in projects"
        :key="p.title"
        class="project"
        :class="{ 'project--clickable': p.description }"
        @click="openProject(p)"
      >
        <div class="project__box">
          <div class="project__box-clip">
            <img
              v-if="p.image"
              :src="p.image"
              class="project__image"
              :style="{ objectPosition: p.imagePosition || 'left center' }"
              alt=""
              draggable="false"
            />
          </div>
          <span class="box-corner box-corner--tl" />
          <span class="box-corner box-corner--tr" />
          <span class="box-corner box-corner--br" />
          <span class="box-corner box-corner--bl" />
        </div>
        <p class="project__title">{{ p.title }}</p>
      </article>
    </div>

    <div v-if="activeProject" class="project-modal" @click.self="closeProject">
      <div class="project-modal__box">
        <button class="project-modal__close" aria-label="Cerrar" @click="closeProject">✕</button>
        <img v-if="activeProject.image" :src="activeProject.image" class="project-modal__image" alt="" draggable="false" />
        <h3 class="project-modal__title">{{ activeProject.title }}</h3>
        <p class="project-modal__text">{{ activeProject.description }}</p>
      </div>
    </div>
  </section>
</template>

<style scoped>
.projects {
  position: relative;
  display: flex;
  align-items: flex-start;
  padding: clamp(16px, 4vh, 56px) clamp(16px, 3.5vw, 48px);
  gap: clamp(16px, 3vw, 48px);
  box-sizing: border-box;
}

/* left column: the project cards that scroll past. The markup puts the cover
   first (so mobile can show it as a title screen before the cards), so on
   desktop this needs an explicit order to land on the left where it belongs —
   flex row layout otherwise follows DOM order and would put it on the right. */
.projects__list {
  order: 1;
  flex: 1 1 46%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: clamp(36px, 7vh, 84px);
  padding-block: 8vh;
}
.project {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 10px;
}
.project--clickable {
  cursor: pointer;
}
.project__box {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 11;
  border: 3px solid #87ddff;
}
/* clips the image to the box without clipping the corner marks, which
   straddle the border and must stay visible outside this inner edge */
.project__box-clip {
  position: absolute;
  inset: 0;
  overflow: hidden;
}
.project__image {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: left center;
  background: #ffffff;
}
.project__title {
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(20px, 2.8vw, 40px);
  text-align: center;
}

.box-corner {
  position: absolute;
  z-index: 1;
  width: 8px;
  height: 8px;
  background: #ffffff;
  border: 1px solid #111;
  box-sizing: border-box;
}
.box-corner--tl { left: -1.5px; top: -1.5px; transform: translate(-50%, -50%); }
.box-corner--tr { right: -1.5px; top: -1.5px; transform: translate(50%, -50%); }
.box-corner--br { right: -1.5px; bottom: -1.5px; transform: translate(50%, 50%); }
.box-corner--bl { left: -1.5px; bottom: -1.5px; transform: translate(-50%, 50%); }

/* right column (desktop): stays pinned while the left scrolls. On mobile this
   becomes its own full-screen view instead — see the media query below. */
.projects__cover {
  order: 2;
  position: sticky;
  top: 0;
  flex: 1 1 52%;
  height: 100svh;
  align-self: flex-start;
  container-type: inline-size;
}

.projects__star {
  position: absolute;
  /* flush with the cover's own corner, so the 50%/50% translate below lands
     the star's center exactly on that corner rather than somewhere near it */
  right: 0;
  bottom: 0;
  transform: translate(50%, 50%);
  transform-origin: center;
  width: min(56vw, 640px);
  aspect-ratio: 1;
  z-index: 1;
  animation: proj-spin 40s linear infinite;
}
@keyframes proj-spin {
  from { transform: translate(50%, 50%) rotate(0deg); }
  to { transform: translate(50%, 50%) rotate(-360deg); }
}

/* both belong to the mobile frame only — the desktop composition is the yellow
   star above and the Hero's own corner marks. Switched on in the media query. */
.projects__burst,
.projects__corner {
  display: none;
}

.projects__word {
  position: absolute;
  /* above the pinned star */
  z-index: 2;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(90px, 27cqw, 420px);
  line-height: 0.85;
}
/* stacked on the right half of the cover, clear of the folder/minecraft
   artwork which stays on the left */
.projects__word--pro { left: 5%; top: 2%; }
.projects__word--yec { left: 56%; top: 36%; }
.projects__word--tos { left: 5%; top: 70%; }

.projects__folder {
  position: absolute;
  right: 10%;
  top: -5%;
  width: 50%;
  height: auto;
  image-rendering: pixelated;
  z-index: 2;
}
.projects__mc {
  position: absolute;
  left: 8%;
  top: 35%;
  width: 30%;
  height: auto;
  image-rendering: pixelated;
  z-index: 3;
}

/* modal shown when a project card with a description is clicked */
.project-modal {
  position: fixed;
  inset: 0;
  z-index: 20;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5vh 5vw;
  background: rgba(0, 0, 0, 0.55);
  animation: project-modal-in 0.2s ease-out;
}
@keyframes project-modal-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
.project-modal__box {
  position: relative;
  width: min(640px, 100%);
  max-height: 85vh;
  overflow-y: auto;
  background: #ffffff;
  border: 3px solid #87ddff;
  box-sizing: border-box;
  padding: clamp(20px, 4vw, 40px);
  animation: project-modal-pop 0.25s cubic-bezier(0.2, 0.8, 0.2, 1);
}
@keyframes project-modal-pop {
  from { transform: scale(0.92); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
.project-modal__close {
  position: absolute;
  top: 12px;
  right: 12px;
  width: 32px;
  height: 32px;
  border: 2px solid #111111;
  background: #ffffff;
  color: #111111;
  font-family: 'DotGothic16', monospace;
  font-size: 16px;
  line-height: 1;
  cursor: pointer;
}
.project-modal__image {
  display: block;
  width: 100%;
  height: auto;
  margin-bottom: 20px;
  border: 2px solid #111111;
}
.project-modal__title {
  margin: 0 0 12px;
  font-family: 'DotGothic16', monospace;
  color: #ff3d00;
  font-size: clamp(24px, 4vw, 36px);
}
.project-modal__text {
  margin: 0;
  font-family: 'DotGothic16', monospace;
  color: #111111;
  font-size: clamp(14px, 1.8vw, 18px);
  line-height: 1.5;
}

/* mobile: the two columns become one. PROYECTOS becomes its own full-screen
   view (cover), then the cards follow below it in normal scroll. */
@media (max-width: 760px) {
  .projects {
    flex-direction: column;
    /* the base align-items: flex-start controls the cross axis, which once
       this is a column means width — it would shrink both children to their
       content instead of filling the screen */
    align-items: stretch;
    gap: 0;
    padding: 0;
    /* the corner star deliberately overflows its column — on a narrow screen
       that alone is enough to make the whole page pannable sideways. Safe to
       clip here because .projects__cover stops being sticky just below, and
       overflow: hidden on an ancestor is what breaks sticky. */
    overflow: hidden;
  }

  .projects__cover {
    order: 1;
    position: relative;
    /* the base rule's flex: 1 1 52% carries a flex-basis, which in a column
       flex container overrides the height property outright — reset it so
       height: 100svh is what actually sizes this element */
    flex: none;
    width: 100%;
    height: 100svh;
    /* the burst runs off the bottom edge by design; without this it would spill
       onto the first project card instead of ending where the view ends */
    overflow: hidden;

    /* One word, broken over three lines — so what separates them is leading,
       not screen real estate. These four numbers are the whole composition.
         --word   font-size. A 3-glyph word inks 1.44em wide (DotGothic16
                  advances 0.5em per Latin glyph) and the frame gives PRO 45.9%
                  of the width: 0.459 / 1.44.
         --ink    0.82 * --word, the measured ink height of one word.
         --pitch  line to line. 1.003em leaves 0.183em of air between the ink of
                  one line and the next — still tight enough that the three read
                  as one word, but no longer crowding. This is the knob: the
                  block stays centred and the drawings follow it, because --top
                  and every element below are stated in multiples of it.
         --top    where PRO's ink starts, placing the 3-line block on the middle
                  of the screen. Block height is 2 * --pitch + --ink. */
    --word: 34cqw;
    --ink: 26.16cqw;
    --pitch: 51cqw;
    --top: calc(50% - (var(--pitch) + var(--ink) / 2));
  }

  /* ---------------------------------------------------------------------
     The Figma frame, rebuilt. Its canvas is about 3:4 where a phone is nearer
     9:19.5, and spreading the vertical positions across that taller screen —
     which is what this block used to do — stretched the gaps between PRO, YEC
     and TOS to three times the height of the letters. The frame's own leading
     is generous because the folder and the creeper sit in the gaps; blown up
     by the phone's proportions it stopped reading as one word.

     So there is one mapping rule now instead of two: everything is a
     proportion of the cover's WIDTH, and every vertical position is stated as
     a multiple of --pitch, the distance from one line of the title to the
     next. The three lines are then set as a block and centred, and the artwork
     keeps the offset from PRO it has in the frame — measured there in frame
     heights and divided by the frame's own 36.4%-of-height line pitch, so
     tightening the leading pulls the whole composition together with it
     instead of leaving the drawings stranded where the letters used to be.

     `left` and `top` are always where the INK should land, and each element
     then pulls itself back by the transparent padding inside its own box.
     Expressed as a percentage of that box the correction scales with the
     element, so none of it needs calc(). All four paddings were measured off
     the assets rather than guessed:
       folder.png     ink starts 25.19% / 25.60% into its 540x500 canvas
       minecraft.png  ink starts 16.17% / 11.94% into its 402x402 canvas
       DotGothic16    ink starts  2.00% /  4.65% into a line-height 0.85 box
       the burst      its hub sits at 56.4% / 47.8% of the polygon's bbox
     --------------------------------------------------------------------- */

  /* the yellow star is the desktop composition; the frame uses the blue one */
  .projects__star { display: none; }

  .projects__word {
    font-size: var(--word);
    transform: translate(-2%, -4.65%);
  }
  /* the horizontal staircase is the frame's, untouched — only the leading
     changed. One --pitch apart, so the three read as one word again. */
  .projects__word--pro { left: 12.9%; top: var(--top); }
  .projects__word--yec { left: 43.2%; top: calc(var(--top) + var(--pitch)); }
  .projects__word--tos { left: 14%; top: calc(var(--top) + 2 * var(--pitch)); }

  .projects__burst {
    display: block;
    position: absolute;
    left: 75%;
    /* frame: hub 82.8% of H, PRO 6.5% — 2.096 line pitches below it */
    top: calc(var(--top) + 2.096 * var(--pitch));
    width: 65%;
    aspect-ratio: 941 / 933;
    /* over the words — in the frame the burst covers the S of TOS */
    z-index: 0;
    transform: translate(-56.4%, -47.8%);
    /* spin about the hub, not the bbox centre, so the point pinned at 81.8% /
       82.8% is the one that stays still */
    transform-origin: 56.4% 47.8%;
    animation: proj-burst-spin 40s linear infinite;
  }
  @keyframes proj-burst-spin {
    from { transform: translate(-56.4%, -47.8%) rotate(0deg); }
    to { transform: translate(-56.4%, -47.8%) rotate(-360deg); }
  }

  /* Behind the letters: with the lines this close the artwork runs under them,
     and the type has to win. z-index 1 puts both under .projects__word's 2. */
  .projects__folder {
    /* the base rule anchors this one by `right` — reset it, or both edges
       would be constrained at once and the width would be ignored */
    right: auto;
    left: 47.4%;
    /* frame: ink top 3.2% of H against PRO's 6.5% — just under a tenth of a
       line pitch above it */
    top: calc(var(--top) - 0.091 * var(--pitch));
    width: 93.9%;
    transform: translate(-25.19%, -25.6%);
    z-index: 1;
  }
  .projects__mc {
    left: 10%;
    /* frame: ink top 37.7% of H — 0.857 pitches below PRO, so it lands between
       YEC and TOS exactly as drawn */
    top: calc(var(--top) + 0.94 * var(--pitch));
    width: 60%;
    transform: translate(-16.17%, -11.94%);
    z-index: 0;
  }

  /* No corner marks here. They used to be switched on for the mobile frame, but
     Projects is the one section that SCROLLS — so marks pinned to its own box
     travelled up the screen with it while the About's identical marks sat still
     underneath, and for the length of the handover there were two sets of dots
     moving against each other. The About's are fixed to the viewport and now
     stay lit for the rest of the page, so the corners are already occupied:
     drawing a second, moving pair on top only fought them. The base rule above
     keeps .projects__corner at display: none. */

  .projects__list {
    order: 2;
    padding: 2vh clamp(16px, 4vw, 24px) 8vh;
    gap: clamp(28px, 5vh, 48px);
  }
  .project__title { font-size: clamp(20px, 5.6vw, 30px); }

  .project-modal { padding: 4vh 5vw; }
  .project-modal__box {
    max-height: 86svh;
    /* keeps a scroll gesture that reaches the end of the modal from chaining
       through to the page behind it */
    overscroll-behavior: contain;
  }
}
</style>
