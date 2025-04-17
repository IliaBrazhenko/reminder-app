<template>
  <div class="input-field">
    <input type="text" v-model="inputValue" placeholder="Введите текст" />
    <p>Введенное значение в верхнем регистре: {{ upperCaseValue }}</p>
    <button @click="sendMessage">Отправить</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'

// Реактивное свойство
const inputValue = ref('')

// Вычисляемое свойство для преобразования в верхний регистр
const upperCaseValue = computed(() => inputValue.value.toUpperCase())

// Функция отправки сообщения
const sendMessage = () => {
  alert(upperCaseValue.value)
}

// Наблюдатель, который сработает при изменении inputValue
watch(inputValue, (newValue, oldValue) => {
  console.log(`Значение изменилось с ${oldValue} на ${newValue}`)
})
</script>

<style scoped>
.input-field {
  text-align: center;
  margin-top: 50px;
}

input {
  padding: 10px;
  border-radius: 5px;
  margin-right: 10px;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>