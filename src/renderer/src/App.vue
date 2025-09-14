<template>
  <div class="text">
    <span class="ts">Kairos</span>
  </div>
  <p class="tip">Just tell me what tasks you have !</p>
  <form class="chat-input" @submit.prevent="onSend">
    <textarea
      ref="ta"
      v-model="message"
      class="chat-textarea"
      :rows="2"
      placeholder="Write a message..."
      @input="autoResize"
      @focus="autoResize"
      @keyup.enter="onSend"
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
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'

const message = ref<string>('')
// If using Vue < 3.5: const ta = ref<HTMLTextAreaElement | null>(null)
// If using Vue 3.5+: useTemplateRef<'ta'> can be inferred; fallback shown below:
const ta = ref<HTMLTextAreaElement | null>(null) // typed template ref [11][16]

const isEmpty = computed<boolean>(() => message.value.trim().length === 0) // typed computed [11]

function autoResize(): void {
  const el = ta.value
  if (!el) return
  el.style.height = 'auto'
  el.style.height = `${el.scrollHeight}px`
}

watch(message, () => autoResize())

onMounted(() => autoResize())

function onSend(): void {
  if (isEmpty.value) return
  // TODO: handle send action (emit, API call, etc.)
  console.log('send:', message.value)
  message.value = ''
  autoResize()
}
</script>

