<template>
  <div
    ref="container"
    class="carousel relative overflow-hidden py-6"
    @mouseenter="onMouseEnter"
    @mouseleave="onMouseLeave"
    @touchstart.passive="onTouchStart"
    @touchend.passive="onTouchEnd"
  >
    <div class="carousel-track flex transition-transform duration-300" :style="trackStyle">
      <div
        v-for="(book, i) in books"
        :key="i"
        class="carousel-item bg-white rounded shadow mx-2 p-3"
        :style="itemStyle"
        :title="book.title"
      >
        <div class="h-40 mb-3 bg-gray-100 rounded overflow-hidden flex items-center justify-center">
          <img v-if="book.image" :src="book.image" :alt="book.title" class="w-full h-full object-cover cursor-pointer" @click="openPdf(book.source, book.title)" />
          <div v-else class="text-sm text-gray-500 px-2">No image</div>
        </div>
        <h3 class="text-sm font-semibold truncate">{{ book.title }}</h3>
        <p v-if="book.source" class="mt-1 text-xs"><a :href="book.source" target="_blank" rel="noopener" class="text-blue-600 hover:underline">Source</a></p>
      </div>
    </div>
    <BookPdfViewer v-if="viewerUrl" :url="viewerUrl" :title="viewerTitle" @close="closePdf" />
    <div class="dots mt-4 flex justify-center gap-2">
      <button
        v-for="p in totalPages"
        :key="p-1"
        @click="index = p-1"
        :class="['w-3 h-3 rounded-full', index === (p-1) ? 'bg-blue-600' : 'bg-gray-300']"
        aria-label="Go to page"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from 'vue'
import BookPdfViewer from './BookPdfViewer.vue'

const props = defineProps({
  books: { type: Array, default: () => [] },
  visible: { type: Number, default: 3 },
  timer: { type: Number, default: 0 }, // seconds; 0 disables auto-rotate
  autoRotate: { type: Boolean, default: true } // enable/disable auto rotation
})

const container = ref(null)
const index = ref(0)
const lastMove = ref(0)
const cooldown = 400 // ms between automatic moves
let rotateInterval = null
const viewerUrl = ref(null)
const viewerTitle = ref('')
const isPaused = ref(false)

function startRotate() {
  stopRotate()
  console.log('[BookCarousell] startRotate timer=', props.timer, 'autoRotate=', props.autoRotate, 'totalPages=', totalPages.value)
  if (!props.autoRotate || viewerUrl.value || isPaused.value) return
  if (props.timer && props.timer > 0 && totalPages.value > 1) {
    rotateInterval = setInterval(() => {
      index.value = (index.value + 1) % totalPages.value
      lastMove.value = Date.now()
      console.log('[BookCarousell] auto-rotate ->', index.value)
    }, props.timer * 1000)
  }
}

function stopRotate() {
  if (rotateInterval) {
    clearInterval(rotateInterval)
    rotateInterval = null
    console.log('[BookCarousell] stopRotate')
  }
}

const totalPages = computed(() => Math.max(1, Math.ceil(props.books.length / props.visible)))

const itemStyle = computed(() => ({
  width: `${100 / props.visible}%`,
  minWidth: `${100 / props.visible}%`,
}))

const trackStyle = computed(() => ({
  transform: `translateX(-${index.value * 100}%)`
}))

function moveNext() {
  const now = Date.now()
  if (now - lastMove.value < cooldown) return
  index.value = Math.min(index.value + 1, totalPages.value - 1)
  lastMove.value = now
}

function movePrev() {
  const now = Date.now()
  if (now - lastMove.value < cooldown) return
  index.value = Math.max(index.value - 1, 0)
  lastMove.value = now
}

// mousemove handlers removed per request

let touchStartX = 0
function onTouchStart(e) { touchStartX = e.touches[0].clientX }
function onTouchEnd(e) {
  const endX = e.changedTouches[0].clientX
  const dx = endX - touchStartX
  if (Math.abs(dx) < 30) return
  if (dx < 0) moveNext()
  else movePrev()
}

function openPdf(url, title) {
  if (!url) return
  isPaused.value = true
  stopRotate()
  viewerUrl.value = url
  viewerTitle.value = title || ''
}

function closePdf() {
  viewerUrl.value = null
  viewerTitle.value = ''
  isPaused.value = false
  if (props.autoRotate) startRotate()
}

function onMouseEnter() {
  isPaused.value = true
  if (props.autoRotate) stopRotate()
}

function onMouseLeave() {
  isPaused.value = false
  if (props.autoRotate && !viewerUrl.value) startRotate()
}

onMounted(() => {
  startRotate()
})

onBeforeUnmount(() => {
  stopRotate()
})

watch(() => props.timer, () => startRotate())
watch(totalPages, () => startRotate())
watch(() => props.autoRotate, (val) => {
  if (val) startRotate()
  else stopRotate()
})
</script>

<style scoped>
.carousel { --gap: 0.5rem }
.carousel-track { gap: var(--gap); }
.carousel-item { box-sizing: border-box }
</style>
