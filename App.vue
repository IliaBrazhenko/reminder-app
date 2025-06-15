<template>
  <div class="app">
    <header class="header">
      <h1>Планируй</h1>
      <p>Управляй событиями легко и эффективно!</p>
    </header>

    <div class="content-wrapper">
      <div class="content">
        <h2>Ваш планировщик событий</h2>

        <!-- Календарь -->
        <div class="calendar">
          <div class="month-header">
            <button @click="prevMonth" class="month-navigation">⬅️</button>
            <h2>{{ monthNames[currentMonth] }} {{ currentYear }}</h2>
            <button @click="nextMonth" class="month-navigation">➡️</button>
          </div>
          <table>
            <thead>
              <tr>
                <td v-for="day in weekDays" :key="day.shortName">{{ day.shortName }}</td>
              </tr>
            </thead>
            <tbody>
              <tr v-for="week in weeks" :key="week.weekNumber">
                <td v-for="day in week.days" :key="day.date || day.dayNumber">
                  <span :class="{ today: day.date && isToday(day.date) }">
                    {{ day.dayNumber }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Форма добавления события -->
        <form @submit.prevent="addEvent" :key="formKey">
          <label for="title-input">Название события:</label>
          <input type="text" id="title-input" v-model="currentEvent.title" placeholder="Название события" required />
          <label for="date-input">Дата и время:</label>
          <input type="datetime-local" id="date-input" v-model="currentEvent.date" required />
          <button type="submit">Добавить событие</button>
        </form>

        <!-- Форма фильтрации и сортировки -->
        <form @submit.prevent="filterEvents">
          <input type="text" v-model="searchQuery" placeholder="Искать события..." />
          <select v-model="sortOrder">
            <option value="asc">По возрастанию</option>
            <option value="desc">По убыванию</option>
          </select>
          <button type="submit">Фильтровать</button>
        </form>

        <!-- Кнопка импорта файла -->
        <div class="import-section">
          <h3>Импорт событий</h3>
          <input 
            type="file" 
            @change="importEventsFromFile" 
            accept=".json,.csv" 
            class="file-input"
          />
          <p>Поддерживаются форматы: .json, .csv</p>
        </div>

        <!-- Список событий -->
        <transition-group tag="ul" name="fade">
          <li v-for="event in filteredEvents" :key="event.id" class="event-item">
            <div class="event-content">
              <span>{{ event.title }} — {{ formatLocalDateTime(event.date) }}</span>
            </div>
            <div class="buttons">
              <button @click="openEditModal(event)" class="edit-btn">Редактировать</button>
              <button @click="deleteEvent(event.id)" class="delete-btn">Удалить</button>
            </div>
          </li>
        </transition-group>

        <!-- Модальное окно редактирования -->
        <div v-if="showEditModal" class="modal">
          <div class="modal-content">
            <h3>Редактировать событие</h3>
            <form @submit.prevent="saveEditedEvent">
              <label for="edit-title">Название события:</label>
              <input type="text" id="edit-title" v-model="editedEvent.title" required />
              <label for="edit-date">Дата и время:</label>
              <input type="datetime-local" id="edit-date" v-model="editedEvent.date" required />
              <button type="submit">Сохранить изменения</button>
              <button @click="closeEditModal">Отмена</button>
            </form>
          </div>
        </div>
      </div>
    </div>

    <footer class="footer">
      <small>© 2025 PlanItUp</small>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onBeforeMount } from 'vue'

// Текущая дата
const now = new Date()

// Год и месяц
const currentYear = ref(now.getFullYear())
const currentMonth = ref(now.getMonth())

// Имена месяцев и дней недели
const monthNames = [
  'Январь', 'Февраль', 'Март', 'Апрель',
  'Май', 'Июнь', 'Июль', 'Август',
  'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'
]
const weekDays = [
  { fullName: 'Понедельник', shortName: 'Пн' },
  { fullName: 'Вторник', shortName: 'Вт' },
  { fullName: 'Среда', shortName: 'Ср' },
  { fullName: 'Четверг', shortName: 'Чт' },
  { fullName: 'Пятница', shortName: 'Пт' },
  { fullName: 'Суббота', shortName: 'Сб' },
  { fullName: 'Воскресенье', shortName: 'Вс' }
]

// События
const events = ref([])
const currentEvent = ref({ title: '', date: '' })
const editedEvent = ref({})
const showEditModal = ref(false)
const searchQuery = ref('')
const sortOrder = ref('asc')
const formKey = ref(0)

// Форматирование даты
const formatLocalDateTime = (isoDate) => {
  const options = { year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric' }
  return new Intl.DateTimeFormat('ru-RU', options).format(new Date(isoDate))
}

// Отфильтрованные события
const filteredEvents = computed(() => {
  let sortedEvents = [...events.value]
  if (sortOrder.value === 'desc') {
    sortedEvents.sort((a, b) => b.date.localeCompare(a.date))
  } else {
    sortedEvents.sort((a, b) => a.date.localeCompare(b.date))
  }
  return sortedEvents.filter(event =>
    event.title.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

// Добавление события
const addEvent = () => {
  if (!currentEvent.value.title.trim()) return alert('Введите название события!')

  const validDate = new Date(currentEvent.value.date)
  if (Number.isNaN(validDate.getTime())) return alert('Некорректная дата!')

  events.value.push({
    id: Date.now(),
    title: currentEvent.value.title,
    date: validDate.toISOString()
  })

  saveEvents()
  resetForm()
}

// Сброс формы
const resetForm = () => {
  currentEvent.value = { title: '', date: '' }
  formKey.value++
}

// Редактирование события
const openEditModal = (event) => {
  editedEvent.value = { ...event }
  showEditModal.value = true
}
const closeEditModal = () => {
  showEditModal.value = false
}
const saveEditedEvent = () => {
  const index = events.value.findIndex(e => e.id === editedEvent.value.id)
  if (index >= 0) {
    events.value[index] = editedEvent.value
    saveEvents()
  }
  closeEditModal()
}

// Удаление события
const deleteEvent = (id) => {
  events.value = events.value.filter(e => e.id !== id)
  saveEvents()
}

// Сохранение в localStorage
const saveEvents = () => {
  localStorage.setItem('events', JSON.stringify(events.value))
}

// Навигация по месяцам
const prevMonth = () => {
  if (currentMonth.value === 0) {
    currentYear.value -= 1
    currentMonth.value = 11
  } else {
    currentMonth.value -= 1
  }
}
const nextMonth = () => {
  if (currentMonth.value === 11) {
    currentYear.value += 1
    currentMonth.value = 0
  } else {
    currentMonth.value += 1
  }
}

// Генерация календаря
const daysInMonth = (year: number, month: number) => {
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const totalDays = lastDay.getDate()
  const startDay = firstDay.getDay()
  const adjustedStart = startDay === 0 ? 7 : startDay

  const weeks = []
  let dayCounter = 1

  for (let i = 0; i < Math.ceil((adjustedStart + totalDays) / 7); i++) {
    const week = { weekNumber: i + 1, days: [] }
    for (let j = 0; j < 7; j++) {
      if (i === 0 && j < adjustedStart - 1) {
        week.days.push({ dayNumber: null })
      } else if (dayCounter <= totalDays) {
        const date = new Date(year, month, dayCounter++)
        week.days.push({ date, dayNumber: date.getDate() })
      } else {
        week.days.push({ dayNumber: null })
      }
    }
    weeks.push(week)
  }

  return weeks
}

// Вычисляем недели
const weeks = computed(() => daysInMonth(currentYear.value, currentMonth.value))

// Проверка, является ли день сегодняшним
const isToday = (date: Date) => {
  if (!date) return false
  const eventDate = new Date(date)
  return (
    eventDate.getFullYear() === now.getFullYear() &&
    eventDate.getMonth() === now.getMonth() &&
    eventDate.getDate() === now.getDate()
  )
}

// Импорт событий из файла
const importEventsFromFile = (event) => {
  const file = event.target.files[0]
  if (!file) return alert('Файл не выбран!')

  const reader = new FileReader()

  reader.onload = (e) => {
    let data = []

    // Если CSV
    if (file.name.endsWith('.csv')) {
      const lines = e.target.result.split('\n').slice(1)
      data = lines.map(line => {
        const [title, date] = line.trim().split(',')
        return {
          id: Date.now() + Math.random(),
          title,
          date,
          notified: false
        }
      })

    // Если JSON
    } else if (file.name.endsWith('.json')) {
      try {
        data = JSON.parse(e.target.result).map(event => ({
          ...event,
          id: event.id || Date.now() + Math.random()
        }))
      } catch (err) {
        console.error('Ошибка парсинга JSON:', err)
        alert('Некорректный формат JSON')
      }

    // Неизвестный формат
    } else {
      return alert('Неподдерживаемый формат файла. Используйте .json или .csv')
    }

    events.value = [...data, ...events.value]
    saveEvents()
    alert(`Загружено ${data.length} событий`)
  }

  reader.onerror = () => {
    alert('Ошибка чтения файла')
  }

  reader.readAsText(file)
}

// Уведомления за 10 минут до события
const requestNotificationPermission = async () => {
  try {
    await Notification.requestPermission()
    console.log('Разрешение на уведомления получено.')
  } catch (err) {
    console.error('Ошибка при запросе разрешения:', err.message)
  }
}

const showNotification = (title, body) => {
  if ('Notification' in window && Notification.permission === 'granted') {
    new Notification(title, { body })
  }
}

const checkNotifications = () => {
  const now = new Date()
  const upcomingEvents = events.value.filter(event => {
    if (!event || !event.date) return false
    const eventDate = new Date(event.date)
    return eventDate > now && eventDate < new Date(now.getTime() + 10 * 60 * 1000)
  })

  upcomingEvents.forEach(event => {
    if (event && event.title) {
      showNotification(event.title, 'Событие скоро начнётся!')
    }
  })
}

// Проверка каждые 5 секунд
setInterval(checkNotifications, 5000)

// При монтировании компонента
onBeforeMount(async () => {
  await requestNotificationPermission()

  const storedEvents = localStorage.getItem('events')
  if (storedEvents) {
    events.value = JSON.parse(storedEvents)
  }
})
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 20px;
  background-color: #f0f5fa;
  color: #333;
  max-width: 1000px;
  margin: 0 auto;
  overflow-x: hidden;
  box-sizing: border-box;
}

.content-wrapper {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
}

.header {
  background-color: #007bff;
  color: white;
  text-align: center;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.calendar table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.calendar th,
.calendar td {
  padding: 10px;
  text-align: center;
}

.today {
  background-color: yellow;
  font-weight: bold;
  color: black;
}

.month-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.month-navigation {
  font-size: 1.5rem;
  background-color: transparent;
  border: none;
  color: #007bff;
  cursor: pointer;
  transition: color 0.3s ease;
}

.month-navigation:hover {
  color: #0056b3;
}

.fade-move,
.fade-enter-active,
.fade-leave-active {
  transition: all 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: scale(1);
}

.event-item {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  margin: 10px 0;
  padding: 10px;
  background-color: #ebf1f9;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  color: #333;
  width: 100%;
  box-sizing: border-box;
}

.event-content {
  margin-bottom: 10px;
  text-align: center;
}

.buttons {
  display: flex;
  justify-content: center;
  gap: 10px;
  box-sizing: border-box;
}

.edit-btn,
.delete-btn {
  flex: 1;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.edit-btn:hover,
.delete-btn:hover {
  background-color: #0056b3;
}

form {
  display: grid;
  gap: 10px;
  margin-bottom: 20px;
  width: 100%;
  box-sizing: border-box;
}

form label {
  font-weight: bold;
}

form input {
  padding: 10px;
  border: 1px solid #ced4da;
  border-radius: 5px;
  transition: border-color 0.3s ease;
}

form input:focus {
  border-color: #007bff;
}

form button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

form button:hover {
  background-color: #0056b3;
}

/* Стили для кнопки импорта файла */
.import-section {
  margin-top: 20px;
  text-align: center;
  width: 100%;
}

.file-input {
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

.file-input:hover {
  background-color: #0056b3;
}

.file-input::file-selector-button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 10px 15px;
  font-size: 1rem;
  cursor: pointer;
}

.file-input::file-selector-button:hover {
  background-color: #0056b3;
}

/* MODAL STYLE */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  box-sizing: border-box;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  width: 300px;
  max-width: 100%;
  text-align: center;
  box-sizing: border-box;
}

.modal-content form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.modal-content button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.modal-content button:hover {
  background-color: #0056b3;
}
</style>