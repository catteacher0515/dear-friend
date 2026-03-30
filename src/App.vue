<template>
  <div id="app">
    <Transition name="scene" mode="out-in">
      <EnvelopeScene
        v-if="scene === 'envelope'"
        key="envelope"
        @open-complete="scene = 'letter'"
      />
      <LetterScene
        v-else-if="scene === 'letter'"
        key="letter"
        :letter="config.letter"
        @continue="scene = 'timeline'"
      />
      <TimelineScene
        v-else-if="scene === 'timeline'"
        key="timeline"
        :items="config.timeline"
      />
    </Transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import EnvelopeScene from './components/EnvelopeScene.vue'
import LetterScene from './components/LetterScene.vue'
import TimelineScene from './components/TimelineScene.vue'

const scene = ref('envelope')
const config = ref({ letter: { recipient: '', content: '' }, timeline: [] })

onMounted(async () => {
  const res = await fetch('/config.json')
  config.value = await res.json()
})
</script>

<style>
.scene-enter-active,
.scene-leave-active {
  transition: opacity 0.6s ease;
}
.scene-enter-from,
.scene-leave-to {
  opacity: 0;
}
</style>
