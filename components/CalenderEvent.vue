<template>
  <div class="calender-event">
    <h2>Ваш планировщик событий</h2>

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
        {{ event.title }} — {{ formatLocalDateTime(event.date) }}
        <button @click="editEvent(event)" class="edit-btn">Редактировать</button>
        <button @click="deleteEvent(event.id)" class="delete-btn">Удалить</button>
      </li>
    </transition-group>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onBeforeMount } from 'vue'

// Массив событий
const events = ref([])

// Новый объект события
const currentEvent = ref({ title: '', date: '' })

// Режим редактирования
const editMode = ref(false)

// Поисковая фраза
const searchQuery = ref('')

// Порядок сортировки
const sortOrder = ref('asc')

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
  margin-bottom: 10px;
  padding: 10px;
  background-color: #ebf1f9;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  color: #333;
}

.event-item button {
  margin-left: 10px;
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
</style>