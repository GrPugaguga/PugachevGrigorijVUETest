<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  initialDate: {
    type: String,
    default: null
  },
  language: {
    type: String,
    default: 'ru',
    validator: (value) => ['ru', 'en'].includes(value)
  }
})

const emit = defineEmits(['dateSelected'])

const locales = {
  ru: {
    months: [
      'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
      'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'
    ],
    weekDays: ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб']
  },
  en: {
    months: [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ],
    weekDays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
  }
}

const currentDate = ref(new Date())
const selectedDate = ref(null)
const viewDate = ref(new Date())

onMounted(() => {
  if (props.initialDate) {
    const parts = props.initialDate.split('-')
    if (parts.length === 3) {
      const date = new Date(parseInt(parts[0]), parseInt(parts[1]) - 1, parseInt(parts[2]))
      if (!isNaN(date.getTime())) {
        selectedDate.value = date
        viewDate.value = new Date(date)
      }
    }
  }
})

const currentMonthName = computed(() => {
  return locales[props.language].months[viewDate.value.getMonth()]
})

const currentYear = computed(() => {
  return viewDate.value.getFullYear()
})

const weekDays = computed(() => {
  return locales[props.language].weekDays
})

const daysInMonth = computed(() => {
  const year = viewDate.value.getFullYear()
  const month = viewDate.value.getMonth()
  return new Date(year, month + 1, 0).getDate()
})

const firstDayOfMonth = computed(() => {
  const year = viewDate.value.getFullYear()
  const month = viewDate.value.getMonth()
  return new Date(year, month, 1).getDay()
})

const calendarDays = computed(() => {
  const days = []
  const totalDays = daysInMonth.value
  const startDay = firstDayOfMonth.value
  
  for (let i = 0; i < startDay; i++) {
    days.push(null)
  }
  
  for (let day = 1; day <= totalDays; day++) {
    days.push(day)
  }
  
  // Always show 42 cells (6 weeks) to maintain fixed size
  while (days.length < 42) {
    days.push(null)
  }
  
  return days
})

const prevMonth = () => {
  const newDate = new Date(viewDate.value)
  newDate.setMonth(newDate.getMonth() - 1)
  viewDate.value = newDate
}

const nextMonth = () => {
  const newDate = new Date(viewDate.value)
  newDate.setMonth(newDate.getMonth() + 1)
  viewDate.value = newDate
}

const selectDate = (day) => {
  if (!day) return
  
  const newDate = new Date(viewDate.value.getFullYear(), viewDate.value.getMonth(), day)
  selectedDate.value = newDate
  
  const year = newDate.getFullYear()
  const month = String(newDate.getMonth() + 1).padStart(2, '0')
  const dayStr = String(day).padStart(2, '0')
  const formattedDate = `${year}-${month}-${dayStr}`
  
  emit('dateSelected', formattedDate)
}

const isToday = (day) => {
  if (!day) return false
  const today = new Date()
  return day === today.getDate() &&
    viewDate.value.getMonth() === today.getMonth() &&
    viewDate.value.getFullYear() === today.getFullYear()
}

const isSelected = (day) => {
  if (!day || !selectedDate.value) return false
  return day === selectedDate.value.getDate() &&
    viewDate.value.getMonth() === selectedDate.value.getMonth() &&
    viewDate.value.getFullYear() === selectedDate.value.getFullYear()
}

const getDayClass = (day) => {
  if (!day) return 'calendar-day-empty'
  let classes = ['calendar-day']
  if (isToday(day)) classes.push('calendar-day-today')
  if (isSelected(day)) classes.push('calendar-day-selected')
  return classes.join(' ')
}
</script>

<template>
  <div class="calendar">
    <div class="calendar-header">
      <button class="calendar-nav-btn" @click="prevMonth">◄</button>
      <div class="calendar-month-year">{{ currentMonthName }} {{ currentYear }}</div>
      <button class="calendar-nav-btn" @click="nextMonth">►</button>
    </div>
    
    <div class="calendar-weekdays">
      <div v-for="weekDay in weekDays" :key="weekDay" class="calendar-weekday">
        {{ weekDay }}
      </div>
    </div>
    
    <div class="calendar-days">
      <div
        v-for="(day, index) in calendarDays"
        :key="index"
        :class="getDayClass(day)"
        @click="selectDate(day)"
      >
        {{ day || '' }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar {
  width: 320px;
  height: 380px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background: #ffffff;
  color: #333;
  position: relative;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.calendar-nav-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  padding: 5px 10px;
  color: #333;
  transition: opacity 0.2s;
}

.calendar-nav-btn:hover {
  opacity: 0.6;
}

.calendar-month-year {
  font-size: 18px;
  font-weight: 500;
}

.calendar-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  margin-bottom: 10px;
}

.calendar-weekday {
  text-align: center;
  font-size: 12px;
  font-weight: 600;
  color: #666;
  padding: 5px;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
}

.calendar-day {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.2s;
  font-size: 14px;
}

.calendar-day:hover {
  background-color: #f0f0f0;
}

.calendar-day-empty {
  aspect-ratio: 1;
}

.calendar-day-today {
  background-color: #e3f2fd;
  font-weight: 600;
}

.calendar-day-selected {
  background-color: #1976d2;
  color: white;
}

.calendar-day-selected:hover {
  background-color: #1565c0;
}
</style>