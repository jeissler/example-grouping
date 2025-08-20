<script setup lang="ts">
import { computed, ref } from 'vue'
import LikeIcon from '@/assets/likeIcon.svg'

// move types and naming
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

enum Sort {
  ASC,
  DESC,
}

const items = ref<ResponseItem[]>([])
const sortDir = ref<Sort>(0)

// experiments with groupBy
const groupedItems = computed(() => {
  const grouped = Object.groupBy(items.value, (item: ResponseItem) => item.user.fullName)

  return Object.entries(grouped).map(([name, items]) => ({
    name,
    items: items?.sort((a, b) => (sortDir.value ? b.likes - a.likes : a.likes - b.likes)) || [],
  })) as GroupedItem[]
})

function toggleSort() {
  sortDir.value = sortDir.value ? 0 : 1
}

try {
  // extract fetch wrapper to base util + composable
  // check for correct headers/data types
  // needs error message handling
  const res = await fetch('https://dummyjson.com/comments')

  if (!res.ok) throw new Error(`Error: ${res.status}`)

  const data = await res.json()

  items.value = data.comments
} catch (err) {
  console.error(err)
}
</script>

<template>
  <div class="mx-auto p-6">
    <section
      v-for="{ name, items } in groupedItems"
      :key="name"
      class="mb-8 bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden"
    >
      <header class="bg-gradient-to-r from-blue-600 to-indigo-600 text-white px-6 py-4">
        <h2 class="text-xl font-semibold">{{ name }}</h2>
        <p class="text-blue-100 text-sm mt-1">
          {{ items.length }} comment{{ items.length > 1 ? 's' : '' }}
        </p>
      </header>

      <div class="p-6 flex flex-col">
        <button
          v-if="items.length > 1"
          class="mb-4 ml-auto px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 flex items-center gap-2 cursor-pointer"
          @click="toggleSort"
        >
          Sort By Likes
        </button>
        <ul class="space-y-4">
          <li
            v-for="{ id, body, likes, postId } in items"
            :key="id"
            class="bg-gray-50 rounded-lg p-4 border-l-4 border-blue-500 hover:shadow-md transition-shadow duration-200"
          >
            <div class="flex items-start justify-between">
              <p class="text-gray-800 leading-relaxed flex-1">{{ body }}</p>
              <div class="flex items-center ml-4 text-blue-600 font-medium">
                <LikeIcon class="w-4 h-4 mr-1" />
                {{ likes }}
              </div>
            </div>
            <div class="mt-3 text-sm text-gray-500">Post ID: {{ postId }}</div>
          </li>
        </ul>
      </div>
    </section>
  </div>
</template>
