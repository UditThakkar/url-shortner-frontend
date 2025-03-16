<template>
  <div class="min-h-screen bg-gray-100 py-12 px-4 sm:px-6 lg:px-8">
    <div class="max-w-2xl mx-auto">
      <div class="bg-white p-6 rounded-lg shadow-md">
        <h1 class="text-3xl font-bold text-gray-900 mb-6">URL Shortener</h1>

        <form @submit.prevent="shortenUrl" class="space-y-6">
          <div>
            <label for="long-url" class="block text-sm font-medium text-gray-700 mb-1">
              Enter Long URL
            </label>
            <input
              v-model="longUrl"
              type="text"
              id="long-url"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
              placeholder="https://example.com/very/long/url"
            />
          </div>

          <button
            type="submit"
            class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            :disabled="isLoading"
            :class="{ 'opacity-50 cursor-not-allowed': isLoading }"
          >
            <svg v-if="isLoading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <span>{{ isLoading ? 'Shortening...' : 'Shorten URL' }}</span>
          </button>

          <div v-if="errorMessage" class="text-red-600 text-sm">
            {{ errorMessage }}
          </div>

          <div v-if="successMessage" class="text-blue-600 text-sm">
            {{ successMessage }}
          </div>

          <div v-if="shortUrl" class="mt-6 p-4 bg-green-50 rounded-md">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-sm font-medium text-green-800">Shortened URL:</p>
                <div class="mt-2 flex items-center">
                  <input
                    :value="shortUrl"
                    type="text"
                    id="short-url"
                    class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                    readonly
                  />
                </div>
              </div>
              <button
                @click.prevent="copyToClipboard"
                class="ml-3 inline-flex items-center px-3 py-1 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
              >
                Copy
              </button>
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>


<script>
import axios from 'axios'

export default {
  data() {
    return {
      longUrl: '',
      shortUrl: '',
      isLoading: false,
      errorMessage: '',
      successMessage: ''
    }
  },
  methods: {
    async shortenUrl() {
      if (!this.longUrl) {
        this.errorMessage = 'Please enter a URL'
        return
      }

      this.isLoading = true
      this.errorMessage = ''
      this.shortUrl = ''
      this.successMessage = ''

      const config = useRuntimeConfig()

      try {
        const response = await axios.post(config.public.backendUrl + '/api/v1/shorten', {
          url: this.longUrl
        })

        this.shortUrl = response.data.shortUrl
      } catch (error) {
        this.errorMessage = error.response?.data?.message || 'Failed to shorten URL'
      } finally {
        this.isLoading = false
      }
    },
    copyToClipboard() {
      if (!this.shortUrl) {
        this.errorMessage = 'No URL to copy';
        return;
      }

      navigator.clipboard.writeText(this.shortUrl)
        .then(() => {
          this.successMessage = 'URL copied to the clipboard successfully'
        })
        .catch(err => {
          this.errorMessage = `Failed to copy URL: ${err.message}`;
          console.error('Failed to copy URL:', err);
        });
    }
  }
}
</script>