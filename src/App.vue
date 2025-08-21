<script setup lang="ts">

//В реальном проекте я бы использовала пакеты для работы с датами и маской инпута
import { ref } from 'vue'
import Calendar from './components/Calendar.vue'

const inputValue = ref<string>('')
const selectedDate = ref<string>('')
const currentLocale = ref('ru')

const isValidDateString = (value: string): boolean => {
  if (!/^\d{4}-\d{2}-\d{2}$/.test(value)) return false
  const [y, m, d] = value.split('-').map(Number)
  const dt = new Date(y, m - 1, d)
  return dt.getFullYear() === y && dt.getMonth() === m - 1 && dt.getDate() === d
}

const maskDateInput = (raw: string): string => {
  const digits = raw.replace(/\D/g, '').slice(0, 8)
  const y = digits.slice(0, 4)
  const m = digits.slice(4, 6)
  const d = digits.slice(6, 8)
  let out = y
  if (m) out += `-${m}`
  if (d) out += `-${d}`
  return out
}

const handleDateSelected = (date: string) => {
  selectedDate.value = date
  inputValue.value = date
}

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  const masked = maskDateInput(target.value)
  inputValue.value = masked
  if (isValidDateString(masked)) {
    selectedDate.value = masked
  }
}

const handlePaste = (event: ClipboardEvent) => {
  event.preventDefault()
  const text = event.clipboardData?.getData('text') ?? ''
  const masked = maskDateInput(text)
  inputValue.value = masked
  if (isValidDateString(masked)) {
    selectedDate.value = masked
  }
}

const toggleLocale = () => {
  currentLocale.value = currentLocale.value === 'ru' ? 'en' : 'ru'
}
</script>

<template>
  <div class="app">
    <main class="app__main">
      <div class="app__input-container">
        <input 
          type="text"
          :value="inputValue"
          @input="handleInput"
          @paste="handlePaste"
          placeholder="ГГГГ-ММ-ДД"
          inputmode="numeric"
          maxlength="10"
          autocomplete="off"
          class="app__date-input"
        />
      </div>

      <div class="app__controls">
        <button class="app__locale-button" @click="toggleLocale">
          Сменить язык ({{ currentLocale }})
        </button>
      </div>

      <div>
        <Calendar 
          v-model="selectedDate"
          :locale="currentLocale"
          @date-selected="handleDateSelected"
        />
      </div>
    </main>
  </div>
</template>

<style scoped>
.app {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  color: #2c3e50;
  background-color: #f8fafc;
  padding: 40px;
}

.app__main {
  width: 400px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.app__input-container {
  width: 100%;
}

.app__controls {
  display: flex;
  justify-content: center;
}

.app__locale-button {
  padding: 10px 16px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.app__locale-button:hover {
  background: #2563eb;
}

.app__date-input {
  width: 100%;
  padding: 12px;
  border: 2px solid #d1d5db;
  border-radius: 6px;
  font-size: 16px;
  box-sizing: border-box;
}

.app__date-input:focus {
  outline: none;
  border-color: #3b82f6;
}
</style>