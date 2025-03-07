<!-- App.vue -->
<template>
  <WelcomeScreen
      :audioSrc="audioFile"
      @experience-started="onExperienceStarted"
  >
    <!-- Здесь размещается основной контент сайта -->
    <div class="site-content">
<!--      <header>-->
<!--        <h1>Название сайта</h1>-->
<!--        &lt;!&ndash; Навигация и другие элементы &ndash;&gt;-->
<!--      </header>-->

      <main>
        <!-- Основное содержимое сайта -->
        <p>Основной контент сайта отображается после взаимодействия пользователя.</p>
      </main>

      <footer>
        <!-- Подвал сайта -->
      </footer>
    </div>
  </WelcomeScreen>
</template>

<script>
import WelcomeScreen from './components/WelcomeScreen.vue';

export default {
  name: 'App',
  components: {
    WelcomeScreen
  },
  data() {
    return {
      audioFile: '/song.mp3'
    }
  },
  methods: {
    onExperienceStarted() {
      console.log('Пользователь начал взаимодействие, аудио запущено');
      // Здесь можно добавить дополнительную логику после начала воспроизведения
    }
  }
}
</script>

<style>
/* Базовые стили */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  transition: background-color 0.5s ease, color 0.5s ease;
}

/* Стили для светлой темы (по умолчанию) */
body {
  background-color: #f9f9f9;
  color: #333;
}

/* Стили для темной темы */
body.dark-mode {
  background-color: #121212;
  color: #e0e0e0;
}

body.dark-mode .site-content {
  color: #e0e0e0;
}

.site-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  font-family: 'Roboto', Arial, sans-serif;
  line-height: 1.6;
  position: relative;
  z-index: 1;
  transition: all 0.5s ease;
}

header {
  margin-bottom: 40px;
  padding-bottom: 20px;
  border-bottom: 1px solid #eaeaea;
  transition: border-color 0.5s ease;
}

body.dark-mode header {
  border-bottom-color: #2c2c2c;
}

header h1 {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0;
  padding: 0;
  transition: color 0.5s ease;
}

body.dark-mode header h1 {
  color: #e0e0e0;
}

main {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

main p {
  font-size: 1.1rem;
  margin: 0;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
}

footer {
  margin-top: 60px;
  padding-top: 20px;
  border-top: 1px solid #eaeaea;
  text-align: center;
  font-size: 0.9rem;
  color: #666;
  transition: border-color 0.5s ease, color 0.5s ease;
}

body.dark-mode footer {
  border-top-color: #2c2c2c;
  color: #a0a0a0;
}

/* Адаптивность для мобильных устройств */
@media (max-width: 768px) {
  .site-content {
    padding: 15px;
  }

  header h1 {
    font-size: 2rem;
  }

  main p {
    font-size: 1rem;
  }
}
</style>