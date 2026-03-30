<template>
  <div class="envelope-scene">
    <!-- 动态背景气泡 -->
    <FloatingBubbles />
    <!-- 彩带（点击时触发） -->
    <Confetti ref="confetti" />

    <div class="envelope-wrapper" @click="startOpen">
      <div class="envelope" :class="{ opening: animationPhase !== 'idle' }">
        <div class="envelope-flap" :class="{ open: animationPhase === 'opening' || animationPhase === 'extracting' || animationPhase === 'done' }"></div>
        <div class="envelope-body"></div>
        <div class="letter-paper" :class="{ extracting: animationPhase === 'extracting' || animationPhase === 'done' }">
          <div class="letter-lines">
            <div class="line" v-for="i in 5" :key="i"></div>
          </div>
        </div>
      </div>

      <p class="hint" v-if="animationPhase === 'idle'">点击打开</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FloatingBubbles from './FloatingBubbles.vue'
import Confetti from './Confetti.vue'

const emit = defineEmits(['open-complete'])
const animationPhase = ref('idle')
const confetti = ref(null)

function startOpen(e) {
  if (animationPhase.value !== 'idle') return

  // 触发彩带爆发
  confetti.value.burst(e.clientX, e.clientY)

  animationPhase.value = 'opening'

  setTimeout(() => {
    animationPhase.value = 'extracting'

    setTimeout(() => {
      animationPhase.value = 'done'
      setTimeout(() => emit('open-complete'), 400)
    }, 800)
  }, 600)
}
</script>

<style scoped>
.envelope-scene {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(ellipse at 60% 40%, #ffe4ec 0%, #fff3e0 40%, #fdf6ec 100%);
  position: relative;
  overflow: hidden;
}

.envelope-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  cursor: pointer;
  user-select: none;
  position: relative;
  z-index: 1;
}

.envelope {
  position: relative;
  width: 360px;
  height: 240px;
  perspective: 800px;
  filter: drop-shadow(0 12px 40px rgba(200, 120, 80, 0.18));
  transition: transform 0.2s ease;
}

.envelope:hover {
  transform: translateY(-4px) scale(1.02);
}

.envelope-body {
  position: absolute;
  inset: 0;
  background: linear-gradient(145deg, #f9e8c8, #f0d4a0);
  border-radius: 6px;
  border: 1px solid #d4b896;
  box-shadow: 0 8px 32px rgba(0,0,0,0.10);
}

.envelope-flap {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 50%;
  background: linear-gradient(160deg, #f5dfa0, #e8c878);
  border: 1px solid #d4b896;
  border-bottom: none;
  border-radius: 6px 6px 0 0;
  transform-origin: top center;
  transform: rotateX(0deg);
  transition: transform 0.6s ease;
  z-index: 2;
  clip-path: polygon(0 0, 100% 0, 50% 100%);
}

.envelope-flap.open {
  transform: rotateX(-180deg);
}

.letter-paper {
  position: absolute;
  left: 10%;
  width: 80%;
  height: 85%;
  background: linear-gradient(180deg, #fffdf7, #fff8ee);
  border: 1px solid #e8d5b0;
  border-radius: 3px;
  bottom: 8px;
  z-index: 1;
  transform: translateY(0);
  transition: transform 0.8s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 10px;
  padding: 16px;
}

.letter-paper.extracting {
  transform: translateY(-130px);
}

.line {
  width: 75%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #d4c4a0, transparent);
  border-radius: 1px;
}

.hint {
  color: #b08860;
  font-size: 14px;
  letter-spacing: 3px;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: translateY(0); }
  50% { opacity: 0.5; transform: translateY(3px); }
}
</style>
