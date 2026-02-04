<template>
  <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/60" @click.self="close">
    <div class="bg-white rounded shadow-lg max-w-3xl w-full max-h-[90vh] overflow-auto p-4">
      <div class="flex justify-between items-center mb-2">
        <div class="font-semibold">{{ props.title }}</div>
        <div class="flex items-center gap-2">
          <button @click="prevPage" :disabled="currentPage<=1" class="px-2 py-1 bg-gray-200 rounded">Prev</button>
          <div class="text-sm">Page</div>
          <input class="w-12 text-center border rounded px-1" v-model.number="currentPage" @change="goToPage" />
          <div class="text-sm">/ {{ totalPages }}</div>
          <button @click="nextPage" :disabled="currentPage>=totalPages" class="px-2 py-1 bg-gray-200 rounded">Next</button>
          <button @click="zoomOut" class="px-2 py-1 bg-gray-200 rounded">-</button>
          <div class="text-sm">{{ Math.round(scale*100) }}%</div>
          <button @click="zoomIn" class="px-2 py-1 bg-gray-200 rounded">+</button>
          <a :href="props.url" target="_blank" rel="noopener" class="px-2 py-1 bg-gray-200 rounded">Open</a>
          <button @click="close" class="px-3 py-1 bg-red-500 text-white rounded hover:bg-red-600">Close</button>
        </div>
      </div>
      <div class="flex justify-center">
        <canvas ref="canvas"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import * as pdfjsLib from 'pdfjs-dist'
// import worker as an asset URL so Vite serves it from the dev server (avoids CORS)
import workerUrl from 'pdfjs-dist/build/pdf.worker.mjs?url'

const props = defineProps({ url: { type: String, required: true }, title: { type: String, default: 'PDF Viewer' } })
const emit = defineEmits(['close'])

const canvas = ref(null)
let pdfDoc = null
const currentPage = ref(1)
const totalPages = ref(0)
const scale = ref(1.2)

function close() {
  emit('close')
}

async function loadPdf() {
  if (!props.url) return
  try {
    pdfjsLib.GlobalWorkerOptions.workerSrc = workerUrl
    const loadingTask = pdfjsLib.getDocument(props.url)
    pdfDoc = await loadingTask.promise
    totalPages.value = pdfDoc.numPages || 1
    currentPage.value = Math.min(Math.max(1, currentPage.value), totalPages.value)
    await renderPage(currentPage.value)
  } catch (err) {
    console.error('PDF render error', err)
  }
}

async function renderPage(pageNum) {
  if (!pdfDoc) return
  try {
    const page = await pdfDoc.getPage(pageNum)
    const viewport = page.getViewport({ scale: scale.value })
    const canvasEl = canvas.value
    const context = canvasEl.getContext('2d')
    canvasEl.height = viewport.height
    canvasEl.width = viewport.width
    const renderContext = { canvasContext: context, viewport }
    const renderTask = page.render(renderContext)
    await renderTask.promise
  } catch (err) {
    console.error('Page render error', err)
  }
}

function prevPage() {
  if (currentPage.value <= 1) return
  currentPage.value -= 1
  renderPage(currentPage.value)
}

function nextPage() {
  if (currentPage.value >= totalPages.value) return
  currentPage.value += 1
  renderPage(currentPage.value)
}

function goToPage() {
  const p = Math.floor(currentPage.value)
  if (p < 1) currentPage.value = 1
  else if (p > totalPages.value) currentPage.value = totalPages.value
  renderPage(currentPage.value)
}

function zoomIn() {
  scale.value = Math.min(scale.value + 0.2, 3)
  renderPage(currentPage.value)
}

function zoomOut() {
  scale.value = Math.max(scale.value - 0.2, 0.4)
  renderPage(currentPage.value)
}

onMounted(() => {
  loadPdf()
})

onBeforeUnmount(() => {
  if (pdfDoc) {
    pdfDoc.destroy()
    pdfDoc = null
  }
})

watch(() => props.url, async () => {
  if (pdfDoc) {
    pdfDoc.destroy()
    pdfDoc = null
  }
  currentPage.value = 1
  scale.value = 1.2
  await loadPdf()
})
</script>

<style scoped>
canvas { max-width: 100%; height: auto }
</style>
