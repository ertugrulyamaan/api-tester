<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage, useClipboard, useTimeAgo } from '@vueuse/core'

// State
const savedRequests = useLocalStorage('saved-requests', [])
const searchQuery = ref('')
const { copy } = useClipboard()

// Emits
const emit = defineEmits(['load-request'])

// Computed
const filteredHistory = computed(() => {
  if (!searchQuery.value) return savedRequests.value
  const query = searchQuery.value.toLowerCase()
  return savedRequests.value.filter(item => 
    item.url.toLowerCase().includes(query) ||
    item.method.toLowerCase().includes(query)
  )
})

// Methods
const formatTime = (timestamp) => {
  const timeAgo = useTimeAgo(timestamp)
  return timeAgo.value
}

const getUrlHost = (url) => {
  try {
    return new URL(url).host
  } catch {
    return url
  }
}

const getUrlPath = (url) => {
  try {
    const urlObj = new URL(url)
    return urlObj.pathname + urlObj.search
  } catch {
    return url
  }
}

const loadRequest = (item) => {
  emit('load-request', item)
}

const copyRequest = (item) => {
  copy(item.url)
}

const removeFromHistory = (index) => {
  if (confirm('Remove this request from saved requests?')) {
    savedRequests.value.splice(index, 1)
  }
}

const clearHistory = () => {
  if (confirm('Are you sure you want to clear all saved requests?')) {
    savedRequests.value = []
  }
}


</script>

<template>
  <div class="request-history">
    <!-- Search and Actions -->
    <div class="sticky top-0 bg-white border-b border-gray-200 p-2 space-y-2">
      <div class="flex items-center gap-2">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search saved requests..."
          class="w-full px-3 py-1.5 text-sm border rounded bg-gray-50 focus:bg-white focus:outline-none focus:ring-1 focus:ring-blue-500"
        />
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-500">
          {{ filteredHistory.length }} saved requests
        </span>
        <div class="flex gap-2">
          <button
            v-if="savedRequests.length"
            @click="clearHistory"
            class="text-xs text-red-500 hover:text-red-600"
          >
            Clear All
          </button>
        </div>
      </div>
    </div>

    <!-- History List -->
    <div class="space-y-1 p-2">
      <div 
        v-for="(item, index) in filteredHistory" 
        :key="index"
        class="group relative rounded hover:bg-gray-50"
      >
        <div class="p-2">
          <!-- Method and URL -->
          <div class="flex items-start gap-2">
            <span 
              :class="{
                'text-green-600': item.method === 'GET',
                'text-blue-600': item.method === 'POST',
                'text-yellow-600': item.method === 'PUT',
                'text-red-600': item.method === 'DELETE',
                'text-purple-600': item.method === 'PATCH'
              }"
              class="font-mono text-sm font-semibold min-w-[45px]"
            >
              {{ item.method }}
            </span>
            <div class="flex-1 min-w-0 cursor-pointer" @click="loadRequest(item)">
              <div class="text-sm text-gray-900 truncate">
                {{ getUrlPath(item.url) }}
              </div>
              <div class="text-xs text-gray-500 truncate">
                {{ getUrlHost(item.url) }}
              </div>
            </div>
          </div>

          <!-- Time -->
          <div class="flex items-center gap-2 mt-1">
            <span class="text-xs text-gray-400">
              Saved {{ formatTime(item.timestamp) }}
            </span>
          </div>
        </div>

        <!-- Quick Actions -->
        <div 
          class="absolute right-1 top-1 hidden group-hover:flex items-center gap-1"
        >
          <button
            @click="copyRequest(item)"
            class="p-1 text-gray-400 hover:text-gray-600"
            title="Copy URL"
          >
            📋
          </button>
          <button
            @click="removeFromHistory(index)"
            class="p-1 text-gray-400 hover:text-red-500"
            title="Remove"
          >
            ×
          </button>
        </div>
      </div>
    </div>

    <!-- Empty State -->
    <div 
      v-if="!savedRequests.length" 
      class="flex flex-col items-center justify-center p-8 text-center text-gray-500"
    >
      <div class="text-3xl mb-2">📋</div>
      <div class="text-sm">No saved requests</div>
      <div class="text-xs mt-1">Click the Save button to add requests here</div>
    </div>

    <!-- No Results -->
    <div 
      v-if="savedRequests.length && !filteredHistory.length" 
      class="flex flex-col items-center justify-center p-8 text-center text-gray-500"
    >
      <div class="text-sm">No matching requests found</div>
    </div>
  </div>
</template>


