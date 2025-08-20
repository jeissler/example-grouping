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

// using reduce you can do the same thing w/o and extra var/block
const groupedItems = computed(() =>
  items.value.reduce((grouped: GroupedItem[], item) => {
    const fullName = item.user.fullName
    const existing = grouped.find((g) => g.name === fullName)
    if (existing) existing.items.push(item)
    else grouped.push({ name: fullName, items: [item] })
    return grouped
  }, []),
)

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
