<script setup>
import { ref } from 'vue'
import Calendar from './components/Calendar.vue'

const selectedDate = ref('')
const currentLang = ref('ru')

const handleDateSelected = (date) => {
  selectedDate.value = date
}

const toggleLanguage = () => {
  currentLang.value = currentLang.value === 'ru' ? 'en' : 'ru'
}
</script>

<template>
  <div class="app">
    <h1>Компонент календаря</h1>
    
    <div class="controls">
      <button @click="toggleLanguage" class="lang-btn">
        Язык: {{ currentLang === 'ru' ? 'Русский' : 'English' }}
      </button>
    </div>
    
    <Calendar 
      :language="currentLang"
      :initial-date="'2025-08-22'"
      @date-selected="handleDateSelected"
    />
    
    <div v-if="selectedDate" class="selected-info">
      <p>Выбранная дата: <strong>{{ selectedDate }}</strong></p>
    </div>
  </div>
</template>

<style scoped>
.app {
  padding: 20px;
  text-align: center;
}

h1 {
  margin-bottom: 30px;
}

.controls {
  margin-bottom: 20px;
}

.lang-btn {
  padding: 8px 16px;
  font-size: 14px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.lang-btn:hover {
  background: #35a372;
}

.selected-info {
  margin-top: 30px;
  padding: 15px;
  background: #ffffff;
  border-radius: 4px;
  max-width: 320px;
  margin-left: auto;
  margin-right: auto;
  color: #333;
}

.selected-info p {
  margin: 0;
  color: #333;
}
</style>
