<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const rootEl = ref<HTMLElement | null>(null)
const canvasEl = ref<HTMLCanvasElement | null>(null)
const revealed = ref(false)

let ctx: CanvasRenderingContext2D | null = null
let isDown = false
let lastCheck = 0
const BRUSH = 42

async function setup() {
  const root = rootEl.value
  const canvas = canvasEl.value
  if (!root || !canvas) return
  try {
    await (document as unknown as { fonts: FontFaceSet }).fonts.ready
  } catch {
    /* ignore */
  }
  const rect = root.getBoundingClientRect()
  if (rect.width < 2 || rect.height < 2) return
  const dpr = window.devicePixelRatio || 1
  canvas.width = Math.round(rect.width * dpr)
  canvas.height = Math.round(rect.height * dpr)
  ctx = canvas.getContext('2d')
  if (!ctx) return
  ctx.scale(dpr, dpr)
  draw(rect.width, rect.height)
}

function draw(w: number, h: number) {
  if (!ctx) return
  ctx.globalCompositeOperation = 'source-over'

  // solid fuchsia base
  const BR = 178
  const BG = 26
  const BB = 112
  ctx.fillStyle = `rgb(${BR},${BG},${BB})`
  ctx.fillRect(0, 0, w, h)

  // dense metallic grain: each speck is the base fuchsia lightened/darkened,
  // which reads as brushed foil rather than a smooth AI gradient
  const n = Math.floor(w * h * 0.75)
  for (let i = 0; i < n; i++) {
    const x = (Math.random() * w) | 0
    const y = (Math.random() * h) | 0
    const f = 0.6 + Math.random() * 0.85 // brightness factor
    if (Math.random() < 0.02) {
      // occasional bright glint
      ctx.fillStyle = 'rgba(255,255,255,0.85)'
    } else {
      const r = Math.min(255, BR * f) | 0
      const g = Math.min(255, BG * f + (f > 1 ? (f - 1) * 160 : 0)) | 0
      const b = Math.min(255, BB * f) | 0
      ctx.fillStyle = `rgb(${r},${g},${b})`
    }
    ctx.fillRect(x, y, 1, 1)
  }

  // soft off-center sheen (subtle, not banded)
  const sheen = ctx.createRadialGradient(w * 0.32, h * 0.22, 0, w * 0.32, h * 0.22, Math.max(w, h) * 0.95)
  sheen.addColorStop(0, 'rgba(255,255,255,0.22)')
  sheen.addColorStop(0.6, 'rgba(255,255,255,0.04)')
  sheen.addColorStop(1, 'rgba(0,0,0,0.1)')
  ctx.fillStyle = sheen
  ctx.fillRect(0, 0, w, h)

  const padX = w * 0.07
  const padY = h * 0.2
  const pw = w - padX * 2
  const ph = h - padY * 2

  // metallic gold "RASPA Y GANA" fitted inside the pill area
  const cy = padY + ph / 2
  const gg = ctx.createLinearGradient(0, cy - ph * 0.32, 0, cy + ph * 0.32)
  gg.addColorStop(0, '#fff2b0')
  gg.addColorStop(0.45, '#f0c93c')
  gg.addColorStop(0.52, '#b9871c')
  gg.addColorStop(1, '#f7dd6e')
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  const label = 'RASPA Y GANA'
  ctx.font = `700 100px 'DotGothic16', monospace`
  const measured = ctx.measureText(label).width || 1
  const fontSize = Math.min(ph * 0.5, ((pw * 0.86) / measured) * 100)
  ctx.font = `700 ${Math.round(fontSize)}px 'DotGothic16', monospace`

  const tx = padX + pw / 2
  // dark outline + drop shadow for contrast against the fuchsia
  ctx.save()
  ctx.shadowColor = 'rgba(60,0,30,0.55)'
  ctx.shadowBlur = fontSize * 0.12
  ctx.shadowOffsetY = fontSize * 0.06
  ctx.lineJoin = 'round'
  ctx.lineWidth = fontSize * 0.16
  ctx.strokeStyle = '#3a0020'
  ctx.strokeText(label, tx, cy)
  ctx.restore()
  ctx.fillStyle = gg
  ctx.fillText(label, tx, cy)
}

function scratchAt(e: PointerEvent) {
  const canvas = canvasEl.value
  if (!ctx || !canvas) return
  const rect = canvas.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  ctx.globalCompositeOperation = 'destination-out'
  ctx.beginPath()
  ctx.arc(x, y, BRUSH, 0, Math.PI * 2)
  ctx.fill()
}

function onDown(e: PointerEvent) {
  isDown = true
  scratchAt(e)
}
function onMove(e: PointerEvent) {
  if (isDown) scratchAt(e)
}
function onUp() {
  isDown = false
  maybeReveal()
}

function maybeReveal() {
  const canvas = canvasEl.value
  if (!ctx || !canvas || revealed.value) return
  const now = Date.now()
  if (now - lastCheck < 300) return
  lastCheck = now
  const { width, height } = canvas
  const data = ctx.getImageData(0, 0, width, height).data
  let clear = 0
  let total = 0
  for (let i = 3; i < data.length; i += 4 * 8) {
    total++
    if (data[i] === 0) clear++
  }
  // require the surface to be fully scratched off
  if (total && clear / total >= 1) revealed.value = true
}

onMounted(() => {
  setup()
  window.addEventListener('pointerup', onUp)
})
onBeforeUnmount(() => window.removeEventListener('pointerup', onUp))
</script>

<template>
  <div ref="rootEl" class="scratch" :class="{ 'is-revealed': revealed }">
    <div class="scratch__under"><slot /></div>
    <canvas
      ref="canvasEl"
      class="scratch__canvas"
      @pointerdown="onDown"
      @pointermove="onMove"
    />
  </div>
</template>

<style scoped>
.scratch {
  position: absolute;
  inset: 0;
  overflow: hidden;
}
.scratch__under {
  position: absolute;
  inset: 0;
}
.scratch__canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  cursor: crosshair;
  touch-action: none;
}
.scratch.is-revealed .scratch__canvas {
  display: none;
}
</style>
