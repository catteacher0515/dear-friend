<template>
  <div class="timeline-scene">
    <div class="timeline-wrapper">
      <h2 class="timeline-title">我们的故事</h2>
      <div class="timeline-track">
        <!-- 竖线 -->
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
  background: #fdf6ec;
}

.timeline-wrapper {
  max-width: 680px;
  margin: 0 auto;
  padding: 64px 24px 80px;
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
