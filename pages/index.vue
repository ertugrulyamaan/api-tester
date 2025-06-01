<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import RequestBuilder from '../components/RequestBuilder.vue'
import ResponseViewer from '../components/ResponseViewer.vue'
import RequestHistory from '../components/RequestHistory.vue'

const response = ref(null)
const responseTime = ref(0)
const selectedRequest = ref(null)
const isSidebarOpen = useLocalStorage('sidebar', true)
const savedRequests = useLocalStorage('saved-requests', [])

const handleResponse = (data) => {
  response.value = data.response
  responseTime.value = data.time
}

const handleSaveRequest = (request) => {
  savedRequests.value.unshift(request)
}

const loadRequest = (request) => {
  selectedRequest.value = request
}

const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}
</script>


<template>
  <div class="min-h-screen bg-gray-50 flex">
    <!-- Sidebar -->
    <div 
      v-show="isSidebarOpen"
      class="w-64 bg-white border-r border-gray-200 flex flex-col"
    >
      <!-- Sidebar Header -->
      <div class="p-4 border-b border-gray-200 flex justify-between items-center">
        <h1 class="text-xl font-bold text-gray-800">API Tester</h1>
        <button
                    @click="toggleSidebar"
                    class="p-2 rounded-full text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition-all duration-300 ease-in-out transform hover:scale-110"
                    v-show="isSidebarOpen"
                    >
                    <svg 
                      class="w-5 h-5"
                      fill="none" 
                      stroke="currentColor" 
                      viewBox="0 0 24 24"
                    >
                      <path 
                        stroke-linecap="round" 
                        stroke-linejoin="round" 
                        stroke-width="2" 
                        d="M15 19l-7-7 7-7"
                      />
                    </svg>
                    </button>
      </div>

      <!-- History List -->
      <div class="flex-1 overflow-y-auto">
        <RequestHistory
          @load-request="loadRequest"
          class="p-2"
        />
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex-1 flex flex-col">
      <!-- Main Header -->
      <header class="bg-white border-b border-gray-200 p-4">
        <div class="flex items-center justify-between">
            <div class="flex">
                <div class="flex items-center gap-4">
                    <button
                    @click="toggleSidebar"
                    class="p-2 rounded-full text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition-all duration-300 ease-in-out transform hover:scale-110"
                    :class="{ 'rotate-180': !isSidebarOpen }"
                    v-show="!isSidebarOpen"
                    >
                    <svg 
                      class="w-5 h-5"
                      fill="none" 
                      stroke="currentColor" 
                      viewBox="0 0 24 24"
                    >
                      <path 
                        stroke-linecap="round" 
                        stroke-linejoin="round" 
                        stroke-width="2" 
                        d="M15 19l-7-7 7-7"
                      />
                    </svg>
                    </button>
                <h2 class="text-lg font-semibold text-gray-700">Request Builder</h2>

            </div>
          </div>
        </div>
      </header>

      <!-- Main Content Area -->
      <div class="flex-1 overflow-y-auto p-6">
        <div class="max-w-6xl mx-auto space-y-6">
          <!-- Request Builder -->
          <div class="bg-white rounded-lg shadow-sm">
            <div class="p-6">
              <RequestBuilder 
                @request-sent="handleResponse"
                @save-request="handleSaveRequest"
                :initial-request="selectedRequest"
              />
            </div>
          </div>

          <!-- Response Viewer -->
          <div class="bg-white rounded-lg shadow-sm">
            <div class="p-6">
              <ResponseViewer
                :response="response"
                :response-time="responseTime"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Add these new styles */
.transition-transform {
  transition-property: transform;
}

.duration-300 {
  transition-duration: 300ms;
}

.transform.rotate-180 {
  transform: rotate(180deg);
}
</style>