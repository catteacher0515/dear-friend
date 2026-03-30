# 生日祝福网站实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个生日祝福静态网站，包含信封拆信交互和时间线两个模块，内容通过 config.json 配置。

**Architecture:** Vue 3 + Vite 单页应用，无路由，通过 App.vue 中的响应式状态控制三个场景（信封、信件、时间线）的切换。信封拆信动画使用纯 CSS 3D 变换实现，场景切换使用 Vue `<Transition>`。

**Tech Stack:** Vue 3, Vite, Swiper.js, CSS 3D transforms

---

## 文件结构

| 文件 | 职责 |
|------|------|
| `src/App.vue` | 场景状态管理（envelope / letter / timeline），加载 config.json，场景切换动画 |
| `src/components/EnvelopeScene.vue` | 信封桌面场景，含信封盖子翻开 + 信纸抽出动画，动画完成后 emit 事件 |
| `src/components/LetterScene.vue` | 信纸展开阅读场景，显示信件正文，底部"继续"按钮 |
| `src/components/TimelineScene.vue` | 时间线容器，渲染所有 TimelineItem |
| `src/components/TimelineItem.vue` | 单条时间线记录，含日期、标题、描述、Swiper 图片轮播 |
| `public/config.json` | 内容配置文件（信件 + 时间线数据） |

---

## Task 1: 初始化项目

**Files:**
- Create: `package.json` (via npm create)
- Create: `public/config.json`
- Create: `src/App.vue`

- [ ] **Step 1: 初始化 Vue 3 + Vite 项目**

```bash
cd C:\dev_workspace\friend
npm create vite@latest . -- --template vue
```

选择 `Vue`，不选 TypeScript。

- [ ] **Step 2: 安装依赖**

```bash
npm install
npm install swiper
```

- [ ] **Step 3: 清理默认文件**

删除以下文件（Vite 模板自带，不需要）：
- `src/components/HelloWorld.vue`
- `src/assets/vue.svg`
- `public/vite.svg`

清空 `src/style.css` 内容，替换为：

```css
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  width: 100%;
  height: 100%;
  overflow: hidden;
  font-family: 'Georgia', serif;
}

#app {
  width: 100%;
  height: 100vh;
}
```

- [ ] **Step 4: 创建 public/config.json**

```json
{
  "letter": {
    "recipient": "小明",
    "content": "亲爱的小明，\n\n今天是你的生日，我想对你说很多很多的话。\n\n祝你生日快乐！"
  },
  "timeline": [
    {
      "date": "2020-06-01",
      "title": "我们第一次见面",
      "description": "那天天气很好，阳光正好。",
      "images": []
    },
    {
      "date": "2021-03-15",
      "title": "一起去了海边",
      "description": "海风很大，我们笑得很开心。",
      "images": []
    }
  ]
}
```

- [ ] **Step 5: Commit**

```bash
git init
git add .
git commit -m "feat: init Vue 3 + Vite project"
```

---

## Task 2: App.vue 场景状态管理与配置加载

**Files:**
- Modify: `src/App.vue`

- [ ] **Step 1: 编写 App.vue**

```vue
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
```

- [ ] **Step 2: 验证页面可以启动**

```bash
npm run dev
```

浏览器打开 `http://localhost:5173`，页面不报错即可（此时组件还未创建，会有报错，下一步解决）。

- [ ] **Step 3: Commit**

```bash
git add src/App.vue src/style.css
git commit -m "feat: add scene state management and config loading"
```

---

## Task 3: EnvelopeScene 组件（信封 + 拆信动画）

**Files:**
- Create: `src/components/EnvelopeScene.vue`

信封动画分三个阶段，通过 `animationPhase` 状态控制：
- `idle`：初始状态，显示信封和点击提示
- `opening`：盖子翻开（CSS 3D rotateX 动画）
- `extracting`：信纸从信封内向上抽出
- `done`：动画完成，emit `open-complete`

- [ ] **Step 1: 创建 EnvelopeScene.vue**

```vue
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
  /* 三角形盖子效果 */
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
```

- [ ] **Step 2: 验证动画流程**

```bash
npm run dev
```

打开浏览器，点击信封，确认：
1. 盖子向上翻开
2. 信纸向上抽出
3. 约 1.8 秒后场景切换（此时 LetterScene 还未实现，会报错，正常）

- [ ] **Step 3: Commit**

```bash
git add src/components/EnvelopeScene.vue
git commit -m "feat: add envelope scene with open animation"
```

---

## Task 4: LetterScene 组件（信件阅读）

**Files:**
- Create: `src/components/LetterScene.vue`

- [ ] **Step 1: 创建 LetterScene.vue**

```vue
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
  background: #fdf6ec;
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
```

- [ ] **Step 2: 验证信件场景**

```bash
npm run dev
```

点击信封完成动画后，确认：
1. 信纸页面出现，有展开动画
2. 显示收件人和信件内容
3. 点击"继续"按钮后场景切换（TimelineScene 还未实现，会报错，正常）

- [ ] **Step 3: Commit**

```bash
git add src/components/LetterScene.vue
git commit -m "feat: add letter reading scene"
```

---

## Task 5: TimelineItem 组件（单条记录）

**Files:**
- Create: `src/components/TimelineItem.vue`

- [ ] **Step 1: 创建 TimelineItem.vue**

```vue
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
```

- [ ] **Step 2: Commit**

```bash
git add src/components/TimelineItem.vue
git commit -m "feat: add timeline item component with swiper"
```

---

## Task 6: TimelineScene 组件（时间线容器）

**Files:**
- Create: `src/components/TimelineScene.vue`

- [ ] **Step 1: 创建 TimelineScene.vue**

```vue
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
```

- [ ] **Step 2: 验证完整流程**

```bash
npm run dev
```

走完完整流程：信封 → 拆信动画 → 信件 → 时间线，确认：
1. 三个场景切换正常，有淡入淡出过渡
2. 时间线显示 config.json 中的两条记录
3. 无控制台报错

- [ ] **Step 3: Commit**

```bash
git add src/components/TimelineScene.vue
git commit -m "feat: add timeline scene, complete full flow"
```

---

## Task 7: 收尾与部署准备

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 更新 index.html 标题**

将 `index.html` 中的 `<title>` 改为：

```html
<title>生日快乐</title>
```

- [ ] **Step 2: 验证生产构建**

```bash
npm run build
npm run preview
```

打开 `http://localhost:4173`，走完完整流程，确认生产构建无问题。

- [ ] **Step 3: 最终 Commit**

```bash
git add index.html
git commit -m "feat: update page title, ready for deployment"
```

- [ ] **Step 4: 部署到 Vercel（手动操作）**

1. 将代码推送到 GitHub 仓库
2. 登录 [vercel.com](https://vercel.com)，导入该仓库
3. Framework Preset 选择 `Vite`，其余默认
4. 点击 Deploy

每次修改 `public/config.json` 并推送代码后，Vercel 会自动重新部署。
