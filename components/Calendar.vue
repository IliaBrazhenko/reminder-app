<template>
  <div class="calendar">
    <table>
      <thead>
        <tr>
          <th colspan="7"><h2>{{ monthNames[currentMonth] }} {{ currentYear }}</h2></th>
        </tr>
        <tr>
          <td v-for="day in weekDays" :key="day.shortName">{{ day.shortName }}</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="week in weeks" :key="week.weekNumber">
          <td v-for="day in week.days" :key="day.date">
            <span :class="{ today: isToday(day.date) }">{{ day.dayNumber }}</span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'

// Текущая дата
const now = new Date()

// Переменные для года и месяца
const currentYear = ref(now.getFullYear())
const currentMonth = ref(now.getMonth())

// Имена месяцев и дней недели
const monthNames = ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь']
const weekDays = [
  { fullName: 'Понедельник', shortName: 'Пн' },
  { fullName: 'Вторник', shortName: 'Вт' },
  { fullName: 'Среда', shortName: 'Ср' },
  { fullName: 'Четверг', shortName: 'Чт' },
  { fullName: 'Пятница', shortName: 'Пт' },
  { fullName: 'Суббота', shortName: 'Сб' },
  { fullName: 'Воскресенье', shortName: 'Вс' },
]

// Генерация недель и дней
const daysInMonth = (year: number, month: number) => {
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const totalDays = lastDay.getDate()
 const weeks = []

  for (let i = 0; i <= Math.ceil(totalDays / 7); i++) {
    const week = { weekNumber: i + 1, days: [] }
    for (let j = 0; j < 7; j++) {
      const dayNumber = i * 7 + j + 1
      if (dayNumber <= totalDays) {
        const date = new Date(year, month, dayNumber)
        week.days.push({ date, dayNumber })
      }
    }
    weeks.push(week)
  }

  return weeks
}

// Недели и дни
const weeks = computed(() => daysInMonth(currentYear.value, currentMonth.value))

// Проверка, является ли день сегодняшним
const isToday = (date: Date) => {
  return (
    date.getFullYear() === now.getFullYear() &&
    date.getMonth() === now.getMonth() &&
    date.getDate() === now.getDate()
  )
}
</script>

<style scoped>
.calendar table {
  width: 100%;
  border-collapse: collapse;
  margin: 0 auto;
}

.calendar th, .calendar td {
  padding: 10px;
  text-align: center;
}

.today {
  background-color: yellow;
  font-weight: bold;
}
</style>