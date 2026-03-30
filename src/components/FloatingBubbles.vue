<template>
  <canvas ref="canvas" class="bubbles-canvas"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref(null)
let animId = null
let bubbles = []

function resize() {
  canvas.value.width = window.innerWidth
  canvas.value.height = window.innerHeight
}

function createBubble() {
  return {
    x: Math.random() * window.innerWidth,
    y: window.innerHeight + Math.random() * 100,
    r: Math.random() * 18 + 4,
    speed: Math.random() * 0.4 + 0.15,
    drift: (Math.random() - 0.5) * 0.3,
    opacity: Math.random() * 0.25 + 0.05,
    hue: Math.random() * 40 + 20, // 暖色调：橙黄粉
  }
}

function init() {
  resize()
  bubbles = Array.from({ length: 35 }, createBubble).map(b => ({
    ...b,
    y: Math.random() * window.innerHeight, // 初始分散在屏幕各处
  }))
}

function draw() {
  const ctx = canvas.value.getContext('2d')
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)

  for (const b of bubbles) {
    ctx.beginPath()
    const grad = ctx.createRadialGradient(b.x - b.r * 0.3, b.y - b.r * 0.3, b.r * 0.1, b.x, b.y, b.r)
    grad.addColorStop(0, `hsla(${b.hue}, 90%, 85%, ${b.opacity * 1.5})`)
    grad.addColorStop(1, `hsla(${b.hue}, 70%, 70%, 0)`)
    ctx.fillStyle = grad
    ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2)
    ctx.fill()

    // 高光
    ctx.beginPath()
    ctx.arc(b.x - b.r * 0.3, b.y - b.r * 0.3, b.r * 0.25, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255,255,255,${b.opacity * 0.8})`
    ctx.fill()

    b.y -= b.speed
    b.x += b.drift

    if (b.y + b.r < 0) {
      Object.assign(b, createBubble())
    }
  }

  animId = requestAnimationFrame(draw)
}

onMounted(() => {
  init()
  draw()
  window.addEventListener('resize', resize)
})

onUnmounted(() => {
  cancelAnimationFrame(animId)
  window.removeEventListener('resize', resize)
})
</script>

<style scoped>
.bubbles-canvas {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}
</style>
