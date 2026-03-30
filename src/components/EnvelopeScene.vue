<template>
  <div class="envelope-scene">
    <div class="scene-bg">
      <!-- 背景装饰占位，后期填充 -->
    </div>

    <div class="envelope-wrapper" @click="startOpen">
      <!-- 信封主体 -->
      <div class="envelope" :class="{ opening: animationPhase !== 'idle' }">
        <!-- 信封盖子 -->
        <div class="envelope-flap" :class="{ open: animationPhase === 'opening' || animationPhase === 'extracting' || animationPhase === 'done' }"></div>
        <!-- 信封正面 -->
        <div class="envelope-body"></div>
        <!-- 信纸 -->
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

const emit = defineEmits(['open-complete'])
const animationPhase = ref('idle')

function startOpen() {
  if (animationPhase.value !== 'idle') return

  // 阶段1：盖子翻开
  animationPhase.value = 'opening'

  setTimeout(() => {
    // 阶段2：信纸抽出
    animationPhase.value = 'extracting'

    setTimeout(() => {
      // 阶段3：完成，通知父组件
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
  background: #fdf6ec;
  position: relative;
}

.scene-bg {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.envelope-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  cursor: pointer;
  user-select: none;
}

.envelope {
  position: relative;
  width: 360px;
  height: 240px;
  perspective: 800px;
}

.envelope-body {
  position: absolute;
  inset: 0;
  background: #f5e6c8;
  border-radius: 4px;
  border: 1px solid #d4b896;
  box-shadow: 0 8px 32px rgba(0,0,0,0.12);
}

.envelope-flap {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 50%;
  background: #edd9a3;
  border: 1px solid #d4b896;
  border-bottom: none;
  border-radius: 4px 4px 0 0;
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
  background: #fffdf7;
  border: 1px solid #e8d5b0;
  border-radius: 2px;
  bottom: 8px;
  z-index: 1;
  transform: translateY(0);
  transition: transform 0.8s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 8px;
  padding: 16px;
}

.letter-paper.extracting {
  transform: translateY(-120px);
}

.line {
  width: 80%;
  height: 2px;
  background: #d4c4a0;
  border-radius: 1px;
}

.hint {
  color: #a08060;
  font-size: 14px;
  letter-spacing: 2px;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
</style>
