<script setup lang="ts">
import { ref } from 'vue'
import folderImg from '../assets/folder.png'
import minecraftImg from '../assets/minecraft.png'
import amadtImg from '../assets/amadt-screenshot.png'
import avranPortfolioImg from '../assets/avransportfolio.png'
import gilbertosImg from '../assets/gilbertos.png'

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
    <!-- left: scrollable list of project cards -->
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

    <!-- right: static panel, sticky so it (and the star) stay pinned to the
         bottom-right corner the whole time the cards scroll -->
    <div class="projects__right">
      <svg class="projects__star" viewBox="0 0 200 200" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
        <polygon :points="starPoints" fill="#F8E10E" />
      </svg>

      <span class="projects__word projects__word--pro">PRO</span>
      <span class="projects__word projects__word--yec">YEC</span>
      <span class="projects__word projects__word--tos">TOS</span>

      <img :src="folderImg" class="projects__folder" alt="" draggable="false" />
      <img :src="minecraftImg" class="projects__mc" alt="" draggable="false" />
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

/* left column: the project cards that scroll past */
.projects__list {
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

/* right column: stays pinned while the left scrolls */
.projects__right {
  position: sticky;
  top: 0;
  flex: 1 1 52%;
  height: 100svh;
  align-self: flex-start;
  container-type: inline-size;
}

.projects__star {
  position: absolute;
  right: calc(-1 * clamp(16px, 3.5vw, 48px));
  bottom: 0;
  transform: translate(45%, 45%);
  transform-origin: center;
  width: min(56vw, 640px);
  aspect-ratio: 1;
  z-index: 1;
  animation: proj-spin 40s linear infinite;
}
@keyframes proj-spin {
  from { transform: translate(45%, 45%) rotate(0deg); }
  to { transform: translate(45%, 45%) rotate(-360deg); }
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
.projects__word--pro { left: 2%; top: 4%; }
.projects__word--yec { left: 44%; top: 38%; }
.projects__word--tos { left: 6%; top: 70%; }

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

@media (max-width: 760px) {
  .projects { flex-direction: column; }
  .projects__right { position: relative; height: 70svh; width: 100%; }
}
</style>
