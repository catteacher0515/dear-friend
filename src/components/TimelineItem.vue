<template>
  <div class="timeline-item">
    <div class="timeline-dot"></div>
    <div class="timeline-content">
      <span class="date">{{ item.date }}</span>
      <h3 class="title">{{ item.title }}</h3>
      <p class="description">{{ item.description }}</p>

      <!-- 图片轮播，仅在有图片时渲染 -->
      <div v-if="item.images && item.images.length > 0" class="swiper-container">
        <swiper
          :modules="modules"
          :pagination="{ clickable: true }"
          :loop="item.images.length > 1"
          class="item-swiper"
        >
          <swiper-slide v-for="(img, i) in item.images" :key="i">
            <img :src="img" :alt="`${item.title} 图片 ${i + 1}`" class="slide-img" />
          </swiper-slide>
        </swiper>
      </div>
    </div>
  </div>
</template>

<script setup>
import { Swiper, SwiperSlide } from 'swiper/vue'
import { Pagination } from 'swiper/modules'
import 'swiper/css'
import 'swiper/css/pagination'

defineProps({
  item: {
    type: Object,
    required: true
  }
})

const modules = [Pagination]
</script>

<style scoped>
.timeline-item {
  display: flex;
  gap: 24px;
  position: relative;
}

.timeline-dot {
  flex-shrink: 0;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #c4a070;
  border: 3px solid #fdf6ec;
  box-shadow: 0 0 0 2px #c4a070;
  margin-top: 6px;
}

.timeline-content {
  flex: 1;
  padding-bottom: 40px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.date {
  font-size: 13px;
  color: #a08060;
  letter-spacing: 1px;
}

.title {
  font-size: 18px;
  color: #3a2510;
  font-weight: 600;
}

.description {
  font-size: 15px;
  line-height: 1.7;
  color: #5a4030;
}

.swiper-container {
  margin-top: 12px;
  border-radius: 8px;
  overflow: hidden;
}

.item-swiper {
  width: 100%;
  max-width: 480px;
  height: 280px;
}

.slide-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
