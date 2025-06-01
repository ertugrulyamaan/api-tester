<script setup>
import { ref, watch } from 'vue'
import { useLocalStorage, useClipboard, useDebounceFn } from '@vueuse/core'

const HTTP_METHODS = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH']

// Props
const props = defineProps({
  initialRequest: {
    type: Object,
    default: null
  }
})

// Emits
const emit = defineEmits(['request-sent', 'save-request'])

// State
const request = ref({
  method: 'GET',
  url: '',
  body: ''
})

const headers = ref([
  { key: 'Content-Type', value: 'application/json' }
])

const isLoading = ref(false)
const error = ref('')
const showUrlSuggestions = ref(false)

// Local Storage
const savedUrls = useLocalStorage('saved-urls', [])

// Clipboard
const { copy } = useClipboard()
const copiedJson = ref(false)
const copiedUrl = ref(false)

// Watch for initial request
watch(() => props.initialRequest, (newRequest) => {
  if (newRequest) {
    request.value = { ...newRequest }
    // Rebuild headers from the request
    if (newRequest.headers) {
      headers.value = Object.entries(newRequest.headers).map(([key, value]) => ({ key, value }))
    }
  }
}, { immediate: true })

// Methods
const addHeader = () => {
  headers.value.push({ key: '', value: '' })
}

const removeHeader = (index) => {
  headers.value.splice(index, 1)
}

const selectUrl = (url) => {
  request.value.url = url
  showUrlSuggestions.value = false
}

const formatJson = () => {
  try {
    const parsed = JSON.parse(request.value.body)
    request.value.body = JSON.stringify(parsed, null, 2)
  } catch (e) {
    error.value = 'Invalid JSON format'
  }
}

const copyBody = () => {
  copy(request.value.body)
  copiedJson.value = true
  setTimeout(() => {
    copiedJson.value = false
  }, 1000)
}

const debouncedSaveRequest = useDebounceFn(() => {
  if (request.value.url && !savedUrls.value.includes(request.value.url)) {
    savedUrls.value = [request.value.url, ...savedUrls.value.slice(0, 9)]
  }
}, 500)

const buildHeaders = () => {
  return headers.value.reduce((acc, { key, value }) => {
    if (key && value) acc[key] = value
    return acc
  }, {})
}

const saveRequest = () => {
  const requestToSave = {
    method: request.value.method,
    url: request.value.url,
    headers: buildHeaders(),
    body: request.value.body,
    timestamp: new Date().toISOString()
  }
  emit('save-request', requestToSave)
}

const copyUrl = () => {
  copy(request.value.url)
  copiedUrl.value = true
  setTimeout(() => {
    copiedUrl.value = false
  }, 1000)
}

const sendRequest = async () => {
  error.value = ''
  isLoading.value = true
  const startTime = Date.now()

  try {
    const headerObject = buildHeaders()
    
    const response = await fetch(request.value.url, {
      method: request.value.method,
      headers: headerObject,
      body: request.value.method !== 'GET' ? request.value.body : undefined
    })

    const data = await response.json()
    const endTime = Date.now()

    // Save to history
    const historyItem = {
      ...request.value,
      headers: headerObject,
      status: response.status,
      timestamp: new Date().toISOString()
    }

    emit('request-sent', {
      response: {
        status: response.status,
        data
      },
      time: endTime - startTime,
      historyItem
    })
  } catch (err) {
    error.value = err.message || 'Request failed'
  } finally {
    isLoading.value = false
  }
}
</script>
<template>
  <div class="request-builder">
    <!-- Method ve URL -->
    <div class="flex gap-2 mb-4">
      <select 
        v-model="request.method"
        class="bg-white border rounded px-3 py-2 text-gray-700 font-medium min-w-[100px]"
      >
        <option v-for="method in HTTP_METHODS" :key="method">{{ method }}</option>
      </select>

      <div class="flex-1 relative">
        <input
          v-model="request.url"
          type="text"
          placeholder="Enter API URL (e.g. https://api.example.com/data)"
          class="w-full border rounded px-3 py-2"
          @input="debouncedSaveRequest"
        />
        <div 
          v-if="savedUrls.length && showUrlSuggestions"
          class="absolute w-full mt-1 bg-white border rounded-lg shadow-lg z-10 max-h-[200px] overflow-y-auto"
        >
          <div
            v-for="url in savedUrls"
            :key="url"
            @click="selectUrl(url)"
            class="px-3 py-2 hover:bg-gray-50 cursor-pointer text-sm"
          >
            {{ url }}
          </div>
        </div>
      </div>

      <button
        @click="saveRequest"
        :disabled="!request.url"
        class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600 disabled:bg-green-300 disabled:cursor-not-allowed"
      >
        Save
      </button>

      <button
        @click="copyUrl"
        v-if="!copiedUrl"
        :disabled="copiedUrl"
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:bg-blue-300 disabled:cursor-not-allowed flex items-center gap-2"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
          <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
        </svg>
      </button>
      <button
        v-if="copiedUrl"
        class="text-sm text-green-500"
      >
        Copied!
      </button>

      <button
        @click="sendRequest"
        :disabled="isLoading || !request.url"
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:bg-blue-300 disabled:cursor-not-allowed flex items-center gap-2"
      >
        <span v-if="isLoading" class="animate-spin">⏳</span>
        <span>{{ isLoading ? 'Sending...' : 'Send' }}</span>
      </button>
    </div>

    <!-- Headers -->
    <div class="mb-4">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-semibold">Headers</h3>
        <button 
          @click="addHeader"
          class="text-sm text-blue-500 hover:text-blue-600"
        >
          + Add Header
        </button>
      </div>
      
      <div class="space-y-2">
        <div 
          v-for="(header, index) in headers" 
          :key="index"
          class="flex gap-2"
        >
          <input
            v-model="header.key"
            placeholder="Key"
            class="flex-1 border rounded px-3 py-2"
          />
          <input
            v-model="header.value"
            placeholder="Value"
            class="flex-1 border rounded px-3 py-2"
          />
          <button 
            @click="removeHeader(index)"
            class="text-red-500 px-2 hover:text-red-600"
          >
            ×
          </button>
        </div>
      </div>
    </div>

    <!-- Request Body -->
    <div class="mb-4">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-semibold">Request Body</h3>
        <div class="flex gap-2">
          <button
            @click="formatJson"
            class="text-sm text-blue-500 hover:text-blue-600"
          >
            Format JSON
          </button>
          <button
            v-if="copiedJson"
            class="text-sm text-green-500"
          >
            Copied!
          </button>
          <button
            v-else
            @click="copyBody"
            class="text-sm text-blue-500 hover:text-blue-600"
          >
            Copy
          </button>
        </div>
      </div>
      
      <textarea
        v-model="request.body"
        :disabled="request.method === 'GET'"
        :placeholder="request.method === 'GET' ? 'Body not allowed for GET requests' : 'Enter request body (JSON)'"
        class="w-full h-48 border rounded p-3 font-mono text-sm disabled:bg-gray-50 disabled:text-gray-500"
      ></textarea>
    </div>

    <!-- Error Message -->
    <div 
      v-if="error"
      class="bg-red-50 border border-red-200 text-red-600 rounded p-3 mb-4"
    >
      {{ error }}
    </div>
  </div>
</template>

