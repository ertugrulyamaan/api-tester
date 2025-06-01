<script setup>
import { ref, computed } from 'vue'
import { useClipboard } from '@vueuse/core'

const props = defineProps({
  response: {
    type: Object,
    default: null
  },
  responseTime: {
    type: Number,
    default: 0
  }
})

// State
const viewMode = ref('Pretty')
const isSearching = ref(false)
const searchQuery = ref('')
const responseEl = ref(null)

// Clipboard
const { copy, copied } = useClipboard()

// Computed
const statusColorClass = computed(() => {
  if (!props.response?.status) return ''
  return {
    'text-green-500': props.response.status < 300,
    'text-yellow-500': props.response.status >= 300 && props.response.status < 400,
    'text-red-500': props.response.status >= 400
  }
})

const formattedResponse = computed(() => {
  if (!props.response) return 'No response yet'
  
  if (viewMode.value === 'Raw') {
    return JSON.stringify(props.response.data)
  }
  
  return JSON.stringify(props.response.data, null, 2)
})

// Methods
const copyResponse = () => {
  copy(formattedResponse.value)
}

const highlightSearch = () => {
  if (!responseEl.value || !searchQuery.value) return
  
  // Implement search highlighting logic here
  const text = responseEl.value.textContent
  const regex = new RegExp(searchQuery.value, 'gi')
  responseEl.value.innerHTML = text.replace(
    regex,
    match => `<mark class="bg-yellow-200">${match}</mark>`
  )
}
</script>

<template>
  <div class="response-viewer">
    <div class="flex justify-between items-center mb-4">
      <h2 class="font-bold">Response</h2>
      <div class="flex items-center gap-4">
        <div class="flex items-center gap-2">
          <span>Time:</span>
          <span>{{ responseTime }}ms</span>
        </div>
        <div class="flex items-center gap-2">
          <span>Status:</span>
          <span :class="statusColorClass">
            {{ response?.status }}
          </span>
        </div>
        <button
          v-if="copied"
          class="text-green-500 text-sm"
        >
          Copied!
        </button>
        <button
          v-else
          @click="copyResponse"
          class="text-blue-500 text-sm hover:text-blue-600"
        >
          Copy Response
        </button>
      </div>
    </div>

    <div class="flex gap-2 mb-4">
      <button
        v-for="view in ['Pretty', 'Raw']"
        :key="view"
        @click="viewMode = view"
        :class="[
          'px-3 py-1 rounded',
          viewMode === view ? 'bg-blue-500 text-white' : 'bg-gray-100'
        ]"
      >
        {{ view }}
      </button>
    </div>
    
    <div class="relative">
      <pre
        ref="responseEl"
        class="bg-gray-100 p-4 rounded overflow-auto max-h-[600px] font-mono text-sm"
      >{{ formattedResponse }}</pre>

      <!-- Search overlay -->
      <div v-if="isSearching" class="absolute top-2 right-2 flex gap-2">
        <input
          v-model="searchQuery"
          type="text"
          class="border rounded px-2 py-1 text-sm"
          placeholder="Search..."
          @input="highlightSearch"
        />
        <button
          @click="isSearching = false"
          class="text-gray-500 hover:text-gray-700"
        >
          ×
        </button>
      </div>
    </div>
  </div>
</template>

