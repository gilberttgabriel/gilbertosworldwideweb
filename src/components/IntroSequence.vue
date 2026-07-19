<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import calmEyes from '../assets/eyes/calm.png'
import doubtEyes from '../assets/eyes/doubt.png'
import seriousEyes from '../assets/eyes/serious.png'

const emit = defineEmits<{ done: [] }>()

type Stage = 'idle' | 'asking' | 'input' | 'submitted' | 'welcome' | 'leaving'
const stage = ref<Stage>('idle')

const currentArt = ref(seriousEyes)

const typedText = ref('')
const inputValue = ref('')
const inputEl = ref<HTMLInputElement | null>(null)

const stageEl = ref<HTMLElement | null>(null)

async function typewrite(text: string, target: (v: string) => void, speed = 55) {
  target('')
  for (let i = 0; i < text.length; i++) {
    target(text.slice(0, i + 1))
    await new Promise((r) => setTimeout(r, speed))
  }
}

async function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms))
}

async function runSequence() {
  await sleep(1000)

  // "quien eres???" typewriting
  stage.value = 'asking'
  await typewrite('quien eres???', (v) => (typedText.value = v))
  await sleep(300)

  // eyebrow raises: swap art
  currentArt.value = doubtEyes
  await sleep(500)

  // input bar appears
  stage.value = 'input'
  await nextTick()
  inputEl.value?.focus()
}

async function submitName() {
  if (stage.value !== 'input' || !inputValue.value.trim()) return
  const name = inputValue.value.trim()
  stage.value = 'submitted'
  currentArt.value = seriousEyes
  await sleep(700)

  currentArt.value = calmEyes
  typedText.value = ''
  stage.value = 'welcome'
  await sleep(200)
  await typewrite(`bienvenido ${name}`, (v) => (typedText.value = v), 45)
  await sleep(800)

  stage.value = 'leaving'
  await sleep(900)
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
        <p v-if="stage === 'asking' || stage === 'input'" class="intro__caption">
          {{ typedText }}<span class="caret">|</span>
        </p>

        <p v-if="stage === 'welcome'" class="intro__caption">
          {{ typedText }}<span class="caret">|</span>
        </p>

        <form v-if="stage === 'input'" class="intro__form" @submit.prevent="submitName">
          <input
            ref="inputEl"
            v-model="inputValue"
            type="text"
            class="intro__input"
            autocomplete="off"
            spellcheck="false"
          />
          <button type="submit" class="intro__submit" aria-label="Enviar">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M4 12.5L9.5 18L20 6" stroke="#ff3d00" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
          </button>
        </form>
      </div>
    </div>

    <div class="intro__squares">
      <span v-for="n in 24" :key="n" class="square"></span>
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
}

.intro__stack {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: min(66vw, calc(52vh * 1112 / 525));
  max-width: 94vw;
  transform: translateY(-6vh);
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

.intro__caption {
  font-family: 'Courier Prime', 'Courier New', monospace;
  font-weight: 700;
  color: #ff3d00;
  font-size: clamp(16px, 2.2vw, 28px);
  white-space: nowrap;
  text-align: center;
}

.caret {
  opacity: 1;
}

.intro__form {
  display: flex;
  align-items: center;
  gap: 12px;
  animation: fade-in 0.35s ease-out;
}

.intro__input {
  width: min(24vw, 240px);
  border: none;
  border-bottom: 2px solid #e5e2da;
  background: transparent;
  padding: 4px 2px 8px;
  font-family: 'Courier Prime', 'Courier New', monospace;
  font-size: clamp(14px, 1.4vw, 17px);
  color: #1a1a1a;
  text-align: center;
  outline: none;
  transition: border-color 0.2s ease;
}
.intro__input::placeholder {
  color: #cfcbc0;
}
.intro__input:focus {
  border-bottom-color: #ff3d00;
}

.intro__submit {
  border: none;
  background: none;
  cursor: pointer;
  line-height: 0;
  padding: 0;
}
.intro__submit svg {
  width: 20px;
  height: 20px;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.intro__squares {
  position: absolute;
  inset: 0;
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  grid-template-rows: repeat(5, 1fr);
  pointer-events: none;
  z-index: 2;
}
.square {
  background: #ff3d00;
  transform: scale(0);
}
.is-leaving .square {
  animation: expand 0.5s cubic-bezier(.7,0,.3,1) forwards;
}
.is-leaving .square:nth-child(4n+1) { animation-delay: 0ms; }
.is-leaving .square:nth-child(4n+2) { animation-delay: 40ms; }
.is-leaving .square:nth-child(4n+3) { animation-delay: 80ms; }
.is-leaving .square:nth-child(4n+4) { animation-delay: 120ms; }

@keyframes expand {
  to { transform: scale(1.02); }
}
</style>
