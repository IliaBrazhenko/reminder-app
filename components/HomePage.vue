<template>
  <div class="home-page">
    <header class="header">
      <h1>PlanItUp</h1>
      <p>Управляй событиями легко и эффективно!</p>
    </header>

    <div class="content">
      <h2>Добро пожаловать на главную страницу!</h2>
      <p>Начните планировать события уже сегодня:</p>
      <ul>
        <li>Создавайте события быстро и просто</li>
        <li>Получайте уведомления о приближающихся мероприятиях</li>
        <li>Управляйте делами в удобное время</li>
      </ul>
    </div>

    <footer class="footer">
      <small>© 2025 PlanItUp</small>
    </footer>
  </div>
</template>

<script setup lang="ts"></script>

<style scoped>
.home-page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 20px;
  background-color: #f0f5fa;
  color: #333;
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
</style>