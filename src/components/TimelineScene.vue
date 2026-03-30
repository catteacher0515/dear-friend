<template>
  <div class="timeline-scene">
    <FloatingBubbles />

    <div class="deco-layer">
      <span class="deco star" style="left:4%;top:8%;animation-delay:0s;font-size:15px">✨</span>
      <span class="deco star" style="left:92%;top:12%;animation-delay:1s;font-size:13px">⭐</span>
      <span class="deco star" style="left:96%;top:45%;animation-delay:0.6s;font-size:16px">✨</span>
      <span class="deco star" style="left:2%;top:55%;animation-delay:1.5s;font-size:14px">⭐</span>
      <span class="deco star" style="left:88%;top:80%;animation-delay:0.3s;font-size:13px">✨</span>
      <span class="deco star" style="left:6%;top:88%;animation-delay:1.8s;font-size:15px">⭐</span>
    </div>

    <div class="timeline-wrapper">
      <h2 class="timeline-title">我们的故事</h2>
      <div class="timeline-track">
        <div class="timeline-line"></div>
        <TimelineItem
          v-for="(item, i) in items"
          :key="i"
          :item="item"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import TimelineItem from './TimelineItem.vue'
import FloatingBubbles from './FloatingBubbles.vue'

defineProps({
  items: {
    type: Array,
    required: true
  }
})
</script>

<style scoped>
.timeline-scene {
  width: 100%;
  height: 100vh;
  overflow-y: auto;
  background: radial-gradient(ellipse at 50% 20%, #ffe4ec 0%, #fff3e0 35%, #fdf6ec 100%);
  position: relative;
}

.deco-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
}

.deco {
  position: absolute;
  user-select: none;
  line-height: 1;
}

.star {
  animation: twinkle 3s ease-in-out infinite;
  opacity: 0.7;
}

@keyframes twinkle {
  0%, 100% { opacity: 0.7; transform: scale(1) rotate(0deg); }
  50%       { opacity: 0.2; transform: scale(0.8) rotate(20deg); }
}

.timeline-wrapper {
  max-width: 680px;
  margin: 0 auto;
  padding: 64px 24px 80px;
  position: relative;
  z-index: 2;
}

.timeline-title {
  font-size: 28px;
  color: #3a2510;
  text-align: center;
  margin-bottom: 56px;
  font-weight: 400;
  letter-spacing: 2px;
}

.timeline-track {
  position: relative;
  padding-left: 32px;
}

.timeline-line {
  position: absolute;
  left: 6px;
  top: 0;
  bottom: 0;
  width: 2px;
  background: linear-gradient(to bottom, #c4a070, #e8d5b0);
}
</style>
