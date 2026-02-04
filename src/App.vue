<template>
   <div class="min-h-screen bg-gray-100">
      <header class="bg-white shadow">
         <div class="max-w-4xl mx-auto p-4">
            <h1 class="text-2xl font-bold">Vue Book Carousell — Examples</h1>
         </div>
      </header>

      <main class="max-w-4xl mx-auto p-6">
         <section>
            <div class="bg-white p-4 rounded shadow">
               <h2 class="text-lg font-semibold mb-3">Books</h2>
               <BookCarousell :books="books" :auto-rotate="true" />
            </div>
         </section>
      </main>
   </div>
</template>

<script setup>
import BookCarousell from './components/BookCarousell.vue'
import examplePdf from './pdf/example.pdf?url'

const samplePdfs = [
   examplePdf,
   'https://file-examples-com.github.io/uploads/2017/10/file-example_PDF_1MB.pdf',
   'https://www.orimi.com/pdf-test.pdf'
]

const books = Array.from({ length: 50 }, (_, i) => ({
   title: `Book ${i + 1}`,
   image: `https://picsum.photos/seed/book${i + 1}/300/400`,
   source: i < samplePdfs.length ? samplePdfs[i] : ''
}))
</script>

<style>
html,body,#app { height: 100%; margin: 0; }
</style>
