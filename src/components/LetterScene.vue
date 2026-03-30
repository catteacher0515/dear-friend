<template>
  <div class="letter-scene">
    <div class="letter-paper">
      <p class="recipient">亲爱的 {{ letter.recipient }}，</p>
      <div class="content">
        <p v-for="(para, i) in paragraphs" :key="i">{{ para }}</p>
      </div>
      <button class="continue-btn" @click="emit('continue')">继续 →</button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  letter: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['continue'])

const paragraphs = computed(() =>
  props.letter.content.split('\n').filter(p => p.trim() !== '')
)
</script>

<style scoped>
.letter-scene {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(ellipse at 40% 60%, #ffe4ec 0%, #fff3e0 40%, #fdf6ec 100%);
  overflow: hidden;
}

.letter-paper {
  width: min(600px, 90vw);
  min-height: 400px;
  background: #fffdf7;
  border: 1px solid #e8d5b0;
  border-radius: 4px;
  box-shadow: 0 8px 40px rgba(0,0,0,0.10);
  padding: 48px 56px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  animation: unfold 0.5s ease;
}

@keyframes unfold {
  from { opacity: 0; transform: scaleY(0.8); }
  to   { opacity: 1; transform: scaleY(1); }
}

.recipient {
  font-size: 18px;
  color: #5a3e28;
  font-style: italic;
}

.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.content p {
  font-size: 16px;
  line-height: 1.8;
  color: #4a3520;
}

.continue-btn {
  align-self: flex-end;
  background: none;
  border: 1px solid #c4a070;
  color: #a08060;
  padding: 10px 24px;
  border-radius: 24px;
  cursor: pointer;
  font-size: 14px;
  letter-spacing: 1px;
  transition: all 0.2s;
}

.continue-btn:hover {
  background: #f5e6c8;
  color: #7a5030;
}
</style>
