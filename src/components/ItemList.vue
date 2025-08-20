<script setup lang="ts">
import { computed, ref } from 'vue'

interface User {
  id: number
  username: string
  fullName: string
}

interface ResponseItem {
  id: number
  body: string
  postId: number
  likes: number
  user: User
}

interface GroupedItem {
  name: string
  items: ResponseItem[]
}

const isLoading = ref(true)
const items = ref<ResponseItem[]>([])

// using map is similar to goto solution, needs template updates
const groupedItems = computed(() => {
  const myMap: Map<string, ResponseItem[]> = new Map()

  for (const item of items.value) {
    const fullName = item.user.fullName
    if (!myMap.has(fullName)) myMap.set(fullName, [])
    myMap.get(fullName)?.push(item)
  }

  return Object.fromEntries(myMap)
})

try {
  // extract fetch wrapper to base util + composable
  // check for correct headers/data types
  // needs message handling
  const res = await fetch('https://dummyjson.com/comments')

  if (!res.ok) throw new Error(`Error: ${res.status}`)

  const data = await res.json()

  items.value = data.comments

  isLoading.value = false
} catch (err) {
  console.error(err)
}
</script>

<template>
  <div>
    <section v-for="user in groupedItems" :key="user.name" class="my-3 bg-indigo-100">
      <header>{{ user.name }}</header>
      <!--
      <ul v-for="item in items" :key="item.id">
        <li>{{ item }}</li>
      </ul>
      -->
      <ul v-for="item in user.items" :key="item.id">
        <li>{{ item.likes }} {{ item.body }}</li>
      </ul>
    </section>
  </div>
</template>

<style scoped></style>
