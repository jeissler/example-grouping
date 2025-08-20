<script setup lang="ts">
import { computed, ref } from 'vue'

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

const isLoading = ref(true)
const items = ref<ResponseItem[]>([])

// experiments with groupBy
const groupedItems = computed(() => {
  const grouped = Object.groupBy(items.value, (item: ResponseItem) => item.user.fullName)

  return Object.entries(grouped).map(([name, items]) => ({
    name,
    items: items?.sort((a, b) => b.likes - a.likes) || [],
  })) as GroupedItem[]
})

try {
  // extract fetch wrapper to base util + composable
  // check for correct headers/data types
  // needs error message handling
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
  <div class="mx-auto p-6">
    <section
      v-for="user in groupedItems"
      :key="user.name"
      class="mb-8 bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden"
    >
      <header class="bg-gradient-to-r from-blue-600 to-indigo-600 text-white px-6 py-4">
        <h2 class="text-xl font-semibold">{{ user.name }}</h2>
        <p class="text-blue-100 text-sm mt-1">{{ user.items.length }} comments</p>
      </header>

      <div class="p-6">
        <ul class="space-y-4">
          <li
            v-for="item in user.items"
            :key="item.id"
            class="bg-gray-50 rounded-lg p-4 border-l-4 border-blue-500 hover:shadow-md transition-shadow duration-200"
          >
            <div class="flex items-start justify-between">
              <p class="text-gray-800 leading-relaxed flex-1">{{ item.body }}</p>
              <div class="flex items-center ml-4 text-blue-600 font-medium">
                <svg class="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
                  <path
                    d="M2 10.5a1.5 1.5 0 113 0v6a1.5 1.5 0 01-3 0v-6zM6 10.333v5.43a2 2 0 001.106 1.79l.05.025A4 4 0 008.943 18h5.416a2 2 0 001.962-1.608l1.2-6A2 2 0 0015.56 8H12V4a2 2 0 00-2-2 1 1 0 00-1 1v.667a4 4 0 01-.8 2.4L6.8 7.933a4 4 0 00-.8 2.4z"
                  />
                </svg>
                {{ item.likes }}
              </div>
            </div>
            <div class="mt-3 text-sm text-gray-500">Post ID: {{ item.postId }}</div>
          </li>
        </ul>
      </div>
    </section>
  </div>
</template>
