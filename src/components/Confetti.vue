<template>
  <canvas ref="canvas" class="confetti-canvas"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref(null)
let animId = null
let particles = []
let running = false

const COLORS = [
  '#ff6b9d', '#ffd93d', '#6bcb77', '#4d96ff',
  '#ff922b', '#cc5de8', '#f06595', '#74c0fc'
]

function resize() {
  canvas.value.width = window.innerWidth
  canvas.value.height = window.innerHeight
}

function createParticle(x, y) {
  const angle = Math.random() * Math.PI * 2
  const speed = Math.random() * 8 + 3
  return {
    x, y,
    vx: Math.cos(angle) * speed,
    vy: Math.sin(angle) * speed - 6,
    gravity: 0.25,
    color: COLORS[Math.floor(Math.random() * COLORS.length)],
    w: Math.random() * 8 + 4,
    h: Math.random() * 4 + 2,
    rotation: Math.random() * Math.PI * 2,
    rotSpeed: (Math.random() - 0.5) * 0.2,
    opacity: 1,
    fade: Math.random() * 0.012 + 0.008,
  }
}

function burst(x, y) {
  if (running) return
  running = true
  particles = Array.from({ length: 120 }, () => createParticle(x, y))
  loop()
}

function loop() {
  const ctx = canvas.value.getContext('2d')
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)

  particles = particles.filter(p => p.opacity > 0)

  for (const p of particles) {
    p.x += p.vx
    p.vy += p.gravity
    p.y += p.vy
    p.rotation += p.rotSpeed
    p.opacity -= p.fade

    ctx.save()
    ctx.translate(p.x, p.y)
    ctx.rotate(p.rotation)
    ctx.globalAlpha = Math.max(0, p.opacity)
    ctx.fillStyle = p.color
    ctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h)
    ctx.restore()
  }

  if (particles.length > 0) {
    animId = requestAnimationFrame(loop)
  } else {
    running = false
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
  }
}

onMounted(() => {
  resize()
  window.addEventListener('resize', resize)
})

onUnmounted(() => {
  cancelAnimationFrame(animId)
  window.removeEventListener('resize', resize)
})

defineExpose({ burst })
</script>

<style scoped>
.confetti-canvas {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 10;
}
</style>
