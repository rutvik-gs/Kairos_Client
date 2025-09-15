<template>
  <div id="app">
    <!-- Settings Button (Top Right) -->
    <button class="settings-btn" title="Settings" @click="showSettings = true">⚙️</button>
    <div class="main-content">
      <div class="text">
        <span class="ts">Kairos</span>
      </div>
      <div class="center-container">
        <p class="tip">
          {{ displayedTip }}
          <span v-if="!typingDone" class="cursor">|</span>
        </p>
      </div>
    </div>

    <form class="chat-input" @submit.prevent="onSend">
      <textarea
        ref="textarea"
        v-model="message"
        class="chat-textarea"
        :rows="2"
        placeholder="Write a message..."
        @keyup.enter.exact="onSend"
        @keydown.enter.shift="onShiftEnter"
      ></textarea>
      <button
        type="submit"
        class="send-btn"
        :disabled="isEmpty"
        aria-label="Send message"
        title="Send"
      >
        ➤
      </button>
    </form>
    <div v-if="showSettings" class="modal-overlay" @click="showSettings = false">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>Settings</h3>
          <button class="close-btn" @click="showSettings = false">×</button>
        </div>
        <div class="modal-body">
          <div class="form-group">
            <label for="api-key">OpenAI API Key</label>
            <input
              id="api-key"
              v-model="settings.apiKey"
              type="password"
              class="form-input"
              placeholder="sk-..."
            />
          </div>
          <div class="form-group">
            <label for="directory">Working Directory</label>
            <div class="directory-input">
              <input
                id="directory"
                v-model="settings.directory"
                type="text"
                class="form-input"
                placeholder="Select a directory..."
                readonly
              />
              <button type="button" class="browse-btn" @click="browseDirectory">Browse</button>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="cancel-btn" @click="showSettings = false">Cancel</button>
          <button class="save-btn" @click="saveSettings">Save</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, reactive, ref, onMounted } from 'vue'
import { useTextareaAutosize } from '@vueuse/core'

// Typing animation logic
const fullTip = 'Just tell me what tasks you have !'
const displayedTip = ref('')
const typingIndex = ref(0)
const typingDone = computed(() => typingIndex.value >= fullTip.length)

function typeTip(): void {
  if (typingIndex.value < fullTip.length) {
    displayedTip.value += fullTip[typingIndex.value]
    typingIndex.value++
    setTimeout(typeTip, 40)
  }
}

// Settings state
const showSettings = ref(false)
const settings = reactive({
  apiKey: '',
  directory: ''
})

// Load settings from localStorage on mount
onMounted(() => {
  typeTip()
  loadSettings()
})

function loadSettings(): void {
  const saved = localStorage.getItem('kairos-settings')
  if (saved) {
    const parsed = JSON.parse(saved)
    settings.apiKey = parsed.apiKey || ''
    settings.directory = parsed.directory || ''
  }
}

function saveSettings(): void {
  localStorage.setItem(
    'kairos-settings',
    JSON.stringify({
      apiKey: settings.apiKey,
      directory: settings.directory
    })
  )
  showSettings.value = false
  console.log('Settings saved:', settings)
}

async function browseDirectory(): Promise<void> {
  try {
    // This will communicate with the main process via IPC
    const selectedPath = await window.api.selectDirectory()
    if (selectedPath) {
      settings.directory = selectedPath
    }
  } catch (error) {
    console.error('Directory selection failed:', error)
  }
}

// Chat functionality
const { textarea, input: message, triggerResize } = useTextareaAutosize()
const isEmpty = computed(() => message.value.trim().length === 0)

function onSend(): void {
  const text = message.value.trim()
  if (!text) return
  console.log('send:', text)
  message.value = ''
  nextTick(() => triggerResize())
}
</script>
