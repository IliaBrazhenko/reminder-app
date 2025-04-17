<template>
  <div class="app">
    <header class="header">
      <h1>PlanItUp</h1>
      <p>Управляй событиями легко и эффективно!</p>
    </header>

    <div class="content-wrapper">
      <div class="content">
        <h2>Ваш планировщик событий</h2>

        <!-- Календарь -->
        <div class="calendar">
          <div class="month-header">
            <button @click="prevMonth" class="month-navigation prev-month">⬅️</button>
            <h2>{{ monthNames[currentMonth] }} {{ currentYear }}</h2>
            <button @click="nextMonth" class="month-navigation next-month">➡️</button>
          </div>

          <table>
            <thead>
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

        <!-- Список событий -->
        <transition-group tag="ul" name="fade">
          <li v-for="event in filteredEvents" :key="event.id" class="event-item">
            <span>{{ event.title }} — {{ formatLocalDateTime(event.date) }}</span>
            <button @click="openEditModal(event)" class="edit-btn">Редактировать</button>
            <button @click="deleteEvent(event.id)" class="delete-btn">Удалить</button>
          </li>
        </transition-group>

        <!-- MODAL WINDOW FOR EDITING EVENTS -->
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
import { ref, reactive, computed, onBeforeMount } from 'vue'

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

// Массив событий
const events = ref([])

// Новый объект события
const currentEvent = ref({ title: '', date: '' })

// Переменные для Modal окна
const showEditModal = ref(false)
const editedEvent = ref({})

// Режим редактирования
const editMode = ref(false)

// Поисковая фраза
const searchQuery = ref('')

// Порядок сортировки
const sortOrder = ref('asc')

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

// Функция форматирования даты и времени в местное время
const formatLocalDateTime = (isoDate) => {
  const options = { year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric' }
  return new Intl.DateTimeFormat('ru-RU', options).format(new Date(isoDate))
}

// Фильтрованные события
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

// Функция добавления события
const addEvent = () => {
  if (currentEvent.value.title.trim() === '') return alert('Введите название события!')

  // Проверяем, что дата валидная
  const validDate = new Date(currentEvent.value.date)
  if (Number.isNaN(validDate.getTime())) {
    return alert('Некорректная дата!')
  }

  events.value.push({ id: Date.now(), title: currentEvent.value.title, date: validDate.toISOString() })
  saveEvents()
  resetForm()
}

// Функция сброса формы
const resetForm = () => {
  currentEvent.value = { title: '', date: '' }
  editMode.value = false
}

// Открыть модальное окно для редактирования события
const openEditModal = (event) => {
  editedEvent.value = { ...event } // Копируем текущее событие для редактирования
  showEditModal.value = true
}

// Закрыть модальное окно
const closeEditModal = () => {
  showEditModal.value = false
}

// Сохранить изменения в событии
const saveEditedEvent = () => {
  const index = events.value.findIndex(e => e.id === editedEvent.value.id)
  if (index >= 0) {
    events.value.splice(index, 1, editedEvent.value)
    saveEvents()
  }
  closeEditModal()
}

// Функция редактирования события
const editEvent = (event) => {
  currentEvent.value = { ...event }
  editMode.value = true
}

// Функция сохранения изменений
const editEventSubmit = () => {
  const index = events.value.findIndex(e => e.id === currentEvent.value.id)
  if (index >= 0) {
    events.value[index] = currentEvent.value
    saveEvents()
    resetForm()
  }
}

// Функция удаления события
const deleteEvent = (id) => {
  events.value = events.value.filter(e => e.id !== id)
  saveEvents()
}

// Функция сохранения событий в localStorage
const saveEvents = () => {
  localStorage.setItem('events', JSON.stringify(events.value))
}

// Смена месяцев в календаре
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

// Получаем разрешение на уведомления
const requestNotificationPermission = async () => {
  try {
    await Notification.requestPermission()
    console.log('Разрешение на уведомления получено.')
  } catch (err) {
    console.error('Ошибка при запросе разрешения:', err)
  }
}

// Отправляем уведомление
const showNotification = (title, body) => {
  if ('Notification' in window && Notification.permission === 'granted') {
    new Notification(title, { body })
  }
}

// Проверка и отправка уведомлений каждые 5 секунд
setInterval(() => {
  checkNotifications()
}, 5000)

// Проверка ближайших событий и отправка уведомлений
const checkNotifications = () => {
  const now = new Date()
  const upcomingEvents = events.value.filter(event => {
    if (!event || !event.date) return false
    const eventDate = new Date(event.date)
    return eventDate > now && eventDate < new Date(now.getTime() + 10 * 60 * 1000) // ближайшие 10 минут
  })

  upcomingEvents.forEach(event => {
    if (event && event.title) {
      showNotification(event.title, 'Событие скоро начнётся!')
    }
  })
}

// Вызываем запрос разрешения при монтировании компонента
onBeforeMount(async () => {
  await requestNotificationPermission()

  // Загрузка событий из localStorage
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
  max-width: 1000px; /* Максимальная ширина страницы */
  margin: 0 auto; /* Центрирование */
  overflow-x: hidden; /* Скрываем горизонтальный скроллинг */
}

.content-wrapper {
  max-width: 800px; /* Максимально допустимая ширина контента */
  margin: 0 auto; /* Центрирование */
  padding: 20px;
  box-sizing: border-box; /* Внутренние отступы учитываются в расчёте ширины */
  overflow-wrap: break-word; /* Принудительно разорвать текст */
}

.header {
  background-color: #007bff;
  color: white;
  text-align: center;
  padding: 20px;
  border-radius: 10px;
}

.content {
  flex-grow: 1;
  text-align: center;
  padding: 20px;
}

.content ul {
  list-style-position: inside;
  padding-left: 0;
}

.footer {
  background-color: #eaeaea;
  padding: 10px;
  text-align: center;
  font-size: 0.8em;
}

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

.month-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.month-navigation {
  font-size: 1.5rem; /* Увеличим размер шрифта */
  background-color: transparent;
  border: none;
  color: #007bff; /* Яркий голубой цвет */
  cursor: pointer;
  user-select: none;
  transition: color 0.3s ease;
}

.month-navigation:hover {
  color: #0056b3; /* Меняем цвет при наведении */
}

.month-navigation.prev-month {
  margin-right: 10px; /* Немного увеличим расстояние между кнопками */
}

.month-navigation.next-month {
  margin-left: 10px; /* То же самое */
}

.calendar h2 {
  font-size: 1.5rem; /* Размер шрифта названия месяца */
  margin: 0;
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
  margin-bottom: 10px;
  padding: 10px;
  background-color: #ebf1f9;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  color: #333;
  max-width: 100%; /* Ограничиваем ширину элемента */
  overflow-wrap: break-word; /* Поведение при длинных словах */
  word-break: break-all; /* Жёсткий перенос текста */
  width: 100%; /* Фиксированная ширина */
}

.event-item span {
  word-break: break-all; /* Дополнительно разрываем длинные слова */
  margin-bottom: 10px;
}

.event-item button {
  display: inline-flex;
  margin: 2px;
  padding: 5px 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.event-item button:hover {
  background-color: #0056b3;
}

form {
  display: grid;
  gap: 10px;
  margin-bottom: 20px;
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

/* MODAL STYLE */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* Полупрозрачный фон */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* Самый верхний слой */
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  width: 300px; /* Явно указываем ширину модального окна */
  max-width: 100%; /* Дополнительное ограничение ширины */
  text-align: center;
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