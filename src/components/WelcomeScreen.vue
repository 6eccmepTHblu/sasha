<!-- WelcomeScreen.vue -->
<template>
  <div>
    <!-- Основной контент сайта -->
    <div v-if="contentVisible" class="main-content dark-theme">
      <slot></slot>

      <!-- Полноэкранный эквалайзер -->
      <div class="fullscreen-equalizer" :class="{ 'active': isPlaying }">
        <div class="equalizer-container">
          <div v-for="i in 20" :key="i" class="equalizer-bar"></div>
        </div>
      </div>

      <!-- Индикатор проигрывания и контроль звука -->
      <div class="audio-controls">
        <button @click="toggleMute" class="mute-button">
          {{ isMuted ? 'Включить звук' : 'Отключить звук' }}
        </button>
        <div class="audio-indicator" :class="{ 'playing': isPlaying }">
          <span v-for="i in 10" :key="i" class="audio-bar"></span>
        </div>
      </div>
    </div>

    <!-- Приветственный экран -->
    <div v-else class="welcome-screen" @click="startExperience">
      <div class="welcome-content">
        <h1>Добро пожаловать</h1>
        <p>Нажмите в любом месте для полного погружения</p>
        <div class="tap-animation">
          <span class="tap-icon"></span>
          <span class="ripple"></span>
        </div>
      </div>
    </div>

    <!-- Скрытый аудио-элемент -->
    <audio ref="audioPlayer" :src="audioSrc" preload="auto" loop></audio>
  </div>
</template>

<script>
export default {
  name: 'WelcomeScreen',
  props: {
    audioSrc: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      contentVisible: false,
      isPlaying: false,
      isMuted: false,
      audioContext: null,
      hasInteracted: false
    }
  },
  mounted() {
    // Предзагрузка аудио
    this.$refs.audioPlayer.load();

    // Добавляем обработчики событий для всего документа
    document.addEventListener('click', this.handleGlobalInteraction);
    document.addEventListener('touchstart', this.handleGlobalInteraction);

    // Создаем AudioContext для определения состояния воспроизведения
    try {
      // Проверка поддержки браузером
      window.AudioContext = window.AudioContext || window.webkitAudioContext;
      this.audioContext = new AudioContext();
    } catch (e) {
      console.warn('Web Audio API не поддерживается в этом браузере:', e);
    }
  },
  beforeUnmount() {
    // Удаляем обработчики при уничтожении компонента
    document.removeEventListener('click', this.handleGlobalInteraction);
    document.removeEventListener('touchstart', this.handleGlobalInteraction);
  },
  methods: {
    // Обработчик глобальных взаимодействий
    handleGlobalInteraction() {
      if (this.hasInteracted) return;

      // Разблокируем AudioContext если он был приостановлен
      if (this.audioContext && this.audioContext.state === 'suspended') {
        this.audioContext.resume();
      }

      this.hasInteracted = true;
    },

    // Запускает воспроизведение и показывает основной контент
    startExperience() {
      // Показываем основной контент
      this.contentVisible = true;

      // Добавляем класс для темной темы к body
      document.body.classList.add('dark-mode');

      // Пытаемся воспроизвести аудио
      this.playAudio();

      // Сообщаем родительскому компоненту о начале
      this.$emit('experience-started');
    },

    // Запускает воспроизведение аудио
    playAudio() {
      const audio = this.$refs.audioPlayer;

      // Пытаемся воспроизвести аудио
      const playPromise = audio.play();

      if (playPromise !== undefined) {
        playPromise
            .then(() => {
              this.isPlaying = true;
              console.log('Аудио успешно воспроизводится');
            })
            .catch(error => {
              console.warn('Автоматическое воспроизведение не удалось:', error);

              // Если автовоспроизведение не удалось, покажем уведомление пользователю
              this.showPlayNotification();
            });
      }
    },

    // Показывает уведомление, если автовоспроизведение не удалось
    showPlayNotification() {
      // Создаем и показываем уведомление пользователю
      const notification = document.createElement('div');
      notification.className = 'play-notification';
      notification.innerHTML = `
        <p>Нажмите здесь, чтобы включить звуковое сопровождение</p>
        <button>Включить звук</button>
      `;

      const button = notification.querySelector('button');
      button.addEventListener('click', () => {
        this.playAudio();
        notification.remove();
      });

      document.body.appendChild(notification);

      // Автоматически скрываем через 10 секунд
      setTimeout(() => {
        if (document.body.contains(notification)) {
          notification.remove();
        }
      }, 10000);
    },

    // Включает/выключает звук
    toggleMute() {
      const audio = this.$refs.audioPlayer;
      audio.muted = !audio.muted;
      this.isMuted = audio.muted;
    }
  }
}
</script>

<style scoped>
/* Стили для темной темы */
:deep(body.dark-mode),
.dark-theme {
  background-color: #121212;
  color: #e0e0e0;
  transition: all 0.3s ease;
}

:deep(body.dark-mode) :deep(header),
.dark-theme :deep(header) {
  border-bottom-color: #2c2c2c;
}

:deep(body.dark-mode) :deep(header h1),
.dark-theme :deep(header h1) {
  color: #e0e0e0;
}

:deep(body.dark-mode) :deep(footer),
.dark-theme :deep(footer) {
  border-top-color: #2c2c2c;
  color: #a0a0a0;
}

/* Полноэкранный эквалайзер */
.fullscreen-equalizer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.85);
  z-index: -1;
  opacity: 0;
  transition: opacity 1s ease;
  overflow: hidden;
  pointer-events: none;
}

.fullscreen-equalizer.active {
  opacity: 1;
}

.equalizer-container {
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  height: 100%;
  width: 100%;
  padding: 0 5%;
  position: absolute;
  bottom: 0;
}

.equalizer-bar {
  width: 3px;
  background: linear-gradient(to top, #4facfe 0%, #00f2fe 100%);
  border-radius: 3px;
  box-shadow: 0 0 8px rgba(0, 242, 254, 0.8);
  height: 5px;
  animation: equalizer-wave 1.5s ease-in-out infinite;
  transform-origin: bottom;
}

@keyframes equalizer-wave {
  0%, 100% {
    height: 5%;
  }
  50% {
    height: 30%;
  }
}

/* Настройка разных высот и задержек для полос эквалайзера */
.equalizer-bar:nth-child(1) { animation-duration: 1.7s; }
.equalizer-bar:nth-child(2) { animation-duration: 2.1s; animation-delay: 0.1s; }
.equalizer-bar:nth-child(3) { animation-duration: 1.9s; animation-delay: 0.2s; }
.equalizer-bar:nth-child(4) { animation-duration: 2.3s; animation-delay: 0.3s; }
.equalizer-bar:nth-child(5) { animation-duration: 1.5s; animation-delay: 0.4s; }
.equalizer-bar:nth-child(6) { animation-duration: 2.2s; animation-delay: 0.5s; }
.equalizer-bar:nth-child(7) { animation-duration: 1.8s; animation-delay: 0.6s; }
.equalizer-bar:nth-child(8) { animation-duration: 2.0s; animation-delay: 0.7s; }
.equalizer-bar:nth-child(9) { animation-duration: 1.6s; animation-delay: 0.8s; }
.equalizer-bar:nth-child(10) { animation-duration: 2.4s; animation-delay: 0.9s; }
.equalizer-bar:nth-child(11) { animation-duration: 1.7s; animation-delay: 0.1s; }
.equalizer-bar:nth-child(12) { animation-duration: 2.1s; animation-delay: 0.2s; }
.equalizer-bar:nth-child(13) { animation-duration: 1.9s; animation-delay: 0.3s; }
.equalizer-bar:nth-child(14) { animation-duration: 2.3s; animation-delay: 0.4s; }
.equalizer-bar:nth-child(15) { animation-duration: 1.5s; animation-delay: 0.5s; }
.equalizer-bar:nth-child(16) { animation-duration: 2.2s; animation-delay: 0.6s; }
.equalizer-bar:nth-child(17) { animation-duration: 1.8s; animation-delay: 0.7s; }
.equalizer-bar:nth-child(18) { animation-duration: 2.0s; animation-delay: 0.8s; }
.equalizer-bar:nth-child(19) { animation-duration: 1.6s; animation-delay: 0.9s; }
.equalizer-bar:nth-child(20) { animation-duration: 2.4s; animation-delay: 1.0s; }

/* Приветственный экран */
.welcome-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #121212;
  background-image:
      radial-gradient(circle at 10% 20%, rgba(30, 144, 255, 0.1) 0%, transparent 50%),
      radial-gradient(circle at 90% 80%, rgba(153, 50, 204, 0.1) 0%, transparent 50%);
  color: white;
  z-index: 1000;
  cursor: pointer;
  overflow: hidden;
  //position: relative;
}

.welcome-screen::before {
  content: "";
  position: absolute;
  width: 200%;
  height: 200%;
  top: -50%;
  left: -50%;
  background-image:
      linear-gradient(to right, transparent, rgba(0, 191, 255, 0.05), transparent),
      linear-gradient(to bottom, transparent, rgba(153, 50, 204, 0.05), transparent);
  animation: aurora 15s linear infinite;
  z-index: -1;
}

@keyframes aurora {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.welcome-content {
  text-align: center;
  max-width: 80%;
}

.welcome-content h1 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 0 0 10px rgba(0, 191, 255, 0.7);
}

.welcome-content p {
  font-size: 1.2rem;
  margin-bottom: 2rem;
  color: rgba(255, 255, 255, 0.8);
}

.tap-animation {
  margin-top: 2rem;
  position: relative;
  width: 60px;
  height: 60px;
  margin-left: auto;
  margin-right: auto;
}

.tap-icon {
  display: block;
  width: 30px;
  height: 30px;
  border: 2px solid #00f2fe;
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 10px rgba(0, 242, 254, 0.8);
}

.ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 30px;
  height: 30px;
  border: 2px solid rgba(0, 242, 254, 0.8);
  border-radius: 50%;
  animation: ripple 1.5s ease-out infinite;
}

@keyframes ripple {
  0% {
    width: 30px;
    height: 30px;
    opacity: 1;
  }
  100% {
    width: 60px;
    height: 60px;
    opacity: 0;
  }
}

/* Аудио контролы */
.audio-controls {
  position: fixed;
  bottom: 20px;
  right: 20px;
  display: flex;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.8);
  padding: 10px 15px;
  border-radius: 30px;
  z-index: 100;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.6), 0 0 10px rgba(0, 191, 255, 0.3);
  backdrop-filter: blur(5px);
  border: 1px solid rgba(30, 144, 255, 0.2);
  transition: all 0.3s ease;
}

.audio-controls:hover {
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.7), 0 0 15px rgba(0, 191, 255, 0.5);
  transform: translateY(-2px);
}

.mute-button {
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #00ffff;
  cursor: pointer;
  padding: 8px 14px;
  font-size: 14px;
  border-radius: 20px;
  transition: all 0.2s ease;
  font-weight: 500;
  text-shadow: 0 0 5px rgba(0, 255, 255, 0.7);
  letter-spacing: 0.5px;
}

.mute-button:hover {
  background-color: rgba(30, 144, 255, 0.3);
  text-shadow: 0 0 8px rgba(0, 255, 255, 1);
}

.audio-indicator {
  display: flex;
  align-items: flex-end;
  height: 30px;
  margin-left: 10px;
  padding: 5px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 15px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5) inset, 0 0 5px rgba(255, 255, 255, 0.1);
}

.audio-bar {
  width: 4px;
  margin: 0 2px;
  background: linear-gradient(to top, #1e90ff, #00bfff, #00ffff);
  height: 5px;
  border-radius: 2px;
  box-shadow: 0 0 3px #00ffff;
  transition: height 0.1s ease;
}

.audio-indicator.playing .audio-bar {
  box-shadow: 0 0 5px #00ffff, 0 0 10px rgba(0, 191, 255, 0.7);
}

/* Создаем 10 полосок эквалайзера вместо 3 */
.audio-indicator.playing .audio-bar:nth-child(1) {
  animation: sound-eq 1.1s ease-in-out infinite alternate;
  height: 15px;
}

.audio-indicator.playing .audio-bar:nth-child(2) {
  animation: sound-eq 1.3s ease-in-out infinite alternate;
  animation-delay: 0.1s;
  height: 22px;
}

.audio-indicator.playing .audio-bar:nth-child(3) {
  animation: sound-eq 0.7s ease-in-out infinite alternate;
  animation-delay: 0.2s;
  height: 10px;
}

.audio-indicator.playing .audio-bar:nth-child(4) {
  animation: sound-eq 0.9s ease-in-out infinite alternate;
  animation-delay: 0.3s;
  height: 20px;
}

.audio-indicator.playing .audio-bar:nth-child(5) {
  animation: sound-eq 1.5s ease-in-out infinite alternate;
  animation-delay: 0.4s;
  height: 25px;
}

.audio-indicator.playing .audio-bar:nth-child(6) {
  animation: sound-eq 0.8s ease-in-out infinite alternate;
  animation-delay: 0.5s;
  height: 18px;
}

.audio-indicator.playing .audio-bar:nth-child(7) {
  animation: sound-eq 1.2s ease-in-out infinite alternate;
  animation-delay: 0.6s;
  height: 12px;
}

.audio-indicator.playing .audio-bar:nth-child(8) {
  animation: sound-eq 1.0s ease-in-out infinite alternate;
  animation-delay: 0.7s;
  height: 23px;
}

.audio-indicator.playing .audio-bar:nth-child(9) {
  animation: sound-eq 1.4s ease-in-out infinite alternate;
  animation-delay: 0.8s;
  height: 16px;
}

.audio-indicator.playing .audio-bar:nth-child(10) {
  animation: sound-eq 0.6s ease-in-out infinite alternate;
  animation-delay: 0.9s;
  height: 19px;
}

@keyframes sound-eq {
  0% {
    height: 5px;
    background: linear-gradient(to top, #1e90ff, #00bfff, #00ffff);
  }
  50% {
    background: linear-gradient(to top, #ff1493, #ff69b4, #ffb6c1);
  }
  100% {
    height: 25px;
    background: linear-gradient(to top, #9932cc, #ba55d3, #da70d6);
  }
}

/* Уведомление о воспроизведении */
.play-notification {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 15px 20px;
  border-radius: 8px;
  text-align: center;
  z-index: 2000;
  max-width: 300px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.7), 0 0 10px rgba(0, 191, 255, 0.3);
  border: 1px solid rgba(30, 144, 255, 0.2);
}

.play-notification button {
  background: linear-gradient(to right, #4facfe 0%, #00f2fe 100%);
  color: white;
  border: none;
  padding: 8px 16px;
  margin-top: 10px;
  border-radius: 4px;
  cursor: pointer;
  box-shadow: 0 0 10px rgba(0, 242, 254, 0.7);
  transition: all 0.2s ease;
}

.play-notification button:hover {
  transform: translateY(-2px);
  box-shadow: 0 0 15px rgba(0, 242, 254, 0.9);
}
</style>