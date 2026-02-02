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
               <BookCarousell :books="books" />
            </div>
         </section>
      </main>
   </div>
</template>

<script setup>
import BookCarousell from './components/BookCarousell.vue'

const books = Array.from({ length: 50 }, (_, i) => ({
   title: `Book ${i + 1}`,
   image: `https://picsum.photos/seed/book${i + 1}/300/400`,
   source: ''
}))
</script>

<style>
html,body,#app { height: 100%; margin: 0; }
</style>
