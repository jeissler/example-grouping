<script setup lang="ts">
import ItemList from '@/components/ItemList.vue'
</script>

<template>
  <header>
    <h1>Page Header</h1>
  </header>
  <main class="flex">
    <Suspense>
      <ItemList />
      <template #fallback>
        <!-- loader component for reuse if using suspense -->
        ...loading
      </template>
    </Suspense>
  </main>
  <footer>
    <p>Page Footer</p>
  </footer>
</template>
