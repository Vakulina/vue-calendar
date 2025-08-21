<template>
  <div class="calendar">
    <div class="calendar__header">
      <button @click="previousMonth" class="calendar__nav-button">
        ◀
      </button>
      <h2 class="calendar__month-year">{{ currentMonthName }} {{ currentYear }}</h2>
      <button @click="nextMonth" class="calendar__nav-button">
        ▶
      </button>
    </div>

    <div class="calendar__weekdays">
      <div 
        v-for="day in weekDays" 
        :key="day" 
        class="calendar__weekday"
      >
        {{ day }}
      </div>
    </div>

    <div class="calendar__grid">
      <div 
        v-for="day in calendarDays" 
        :key="day.key"
        @click="selectDate(day)"
        :class="[
          'calendar__day',
          {
            'calendar__day_other-month': !day.isCurrentMonth,
            'calendar__day_selected': day.isSelected,
            'calendar__day_today': day.isToday
          }
        ]"
      >
        {{ day.dayNumber }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

interface CalendarDay {
  key: string
  dayNumber: number
  date: Date
  isCurrentMonth: boolean
  isSelected: boolean
  isToday: boolean
}

interface LocaleData {
  months: string[]
  weekdays: string[]
}

interface Props {
  modelValue?: string
  locale?: string
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  locale: 'ru'
})

const emit = defineEmits<{
  'update:modelValue': [value: string]
  'date-selected': [value: string]
}>()

const currentDate = ref(new Date())
const selectedDate = ref<Date | null>(null)

const locales: Record<string, LocaleData> = {
  ru: {
    months: [
      'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
      'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'
    ],
    weekdays: ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб']
  },
  en: {
    months: [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ],
    weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
  }
}

const currentYear = computed(() => currentDate.value.getFullYear())
const currentMonth = computed(() => currentDate.value.getMonth())

const currentMonthName = computed(() => {
  const localeData = locales[props.locale] 
  return localeData.months[currentMonth.value]
})

const weekDays = computed(() => {
  const localeData = locales[props.locale] 
  return localeData.weekdays
})

const calendarDays = computed(() => {
  const year = currentYear.value
  const month = currentMonth.value
  
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  
  const firstDayOfWeek = firstDay.getDay()
  const daysInMonth = lastDay.getDate()
  
  const days: CalendarDay[] = []
  
  const prevMonth = new Date(year, month, 0)
  const daysInPrevMonth = prevMonth.getDate()
  
  for (let i = firstDayOfWeek - 1; i >= 0; i--) {
    const dayNumber = daysInPrevMonth - i
    const date = new Date(year, month - 1, dayNumber)
    days.push({
      key: `prev-${dayNumber}`,
      dayNumber,
      date,
      isCurrentMonth: false,
      isSelected: isSameDate(date, selectedDate.value),
      isToday: isSameDate(date, new Date())
    })
  }
  
  for (let day = 1; day <= daysInMonth; day++) {
    const date = new Date(year, month, day)
    days.push({
      key: `current-${day}`,
      dayNumber: day,
      date,
      isCurrentMonth: true,
      isSelected: isSameDate(date, selectedDate.value),
      isToday: isSameDate(date, new Date())
    })
  }
  
  const remainingDays = 42 - days.length
  for (let day = 1; day <= remainingDays; day++) {
    const date = new Date(year, month + 1, day)
    days.push({
      key: `next-${day}`,
      dayNumber: day,
      date,
      isCurrentMonth: false,
      isSelected: isSameDate(date, selectedDate.value),
      isToday: isSameDate(date, new Date())
    })
  }
  
  return days
})

const isSameDate = (date1: Date, date2: Date | null): boolean => {
  if (!date2) return false
  return date1.getDate() === date2.getDate() &&
         date1.getMonth() === date2.getMonth() &&
         date1.getFullYear() === date2.getFullYear()
}

const formatDate = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const parseDate = (dateString: string): Date | null => {
  if (!dateString) return null
  const date = new Date(dateString)
  return isNaN(date.getTime()) ? null : date
}

const selectDate = (day: CalendarDay) => {
  selectedDate.value = day.date
  const formattedDate = formatDate(day.date)
  emit('update:modelValue', formattedDate)
  emit('date-selected', formattedDate)
}

const previousMonth = () => {
  currentDate.value = new Date(currentYear.value, currentMonth.value - 1, 1)
}

const nextMonth = () => {
  currentDate.value = new Date(currentYear.value, currentMonth.value + 1, 1)
}

const initializeDate = () => {
  if (props.modelValue) {
    const parsedDate = parseDate(props.modelValue)
    if (parsedDate) {
      selectedDate.value = parsedDate
      currentDate.value = new Date(parsedDate.getFullYear(), parsedDate.getMonth(), 1)
    }
  } else {
    const today = new Date()
    selectedDate.value = today
    currentDate.value = new Date(today.getFullYear(), today.getMonth(), 1)
  }
}

watch(() => props.modelValue, () => {
  initializeDate()
})

watch(() => props.locale, () => {
})

onMounted(() => {
  initializeDate()
})
</script>

<style scoped>
.calendar {
  width: 100%;
  max-width: 400px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.5);
}

.calendar__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
}

.calendar__month-year {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #374151;
}

.calendar__nav-button {
  background: none;
  border: none;
  font-size: 16px;
  color: #6b7280;
  cursor: pointer;
  padding: 8px;
  border-radius: 4px;
  transition: all 0.2s;
}

.calendar__nav-button:hover {
  background-color: #f3f4f6;
  color: #374151;
}

.calendar__weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 1px;
  background-color: #f9fafb;
}

.calendar__weekday {
  padding: 12px 8px;
  text-align: center;
  font-size: 12px;
  font-weight: 600;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.calendar__grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 1px;
  background-color: #f9fafb;
}

.calendar__day {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  color: #374151;
  cursor: pointer;
  transition: all 0.2s;
  background-color: white;
  border: 1px solid transparent;
}

.calendar__day:hover {
  background-color: #dcdceb;
}

.calendar__day_other-month {
  color: #d1d5db;
}

.calendar__day_selected {
  background-color: #3b82f6;
  color: white;
  border-color: #3b82f6;
}

.calendar__day_selected:hover {
  background-color: #3b82f6;
}

.calendar__day_today {
  border: 2px solid #3b82f6;
  font-weight: 600;
}
</style>
