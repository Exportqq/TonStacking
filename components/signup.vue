<template>
    <div>
        <div class="signup-backround">
            <div class="navigation">
                <div style="width: 327px; margin: 60px 0px 0px 0px;">
                    <img src="public/logo.svg" alt="Logo">
                </div>
                <div class="signup-navigation" style="margin: 38px 0px 31px 0px;">
                    <p class="sign-up-txt">Get Started now</p>
                    <p class="sign-up-comment">Create an account or sign in to learn more about the world of staking</p>
                </div>
            </div>
            <div class="signup-block">
                <div>
                    <div class="choose">
                        <div class="slider" :class="{ 'slide-right': !isLoginActive }"></div>
                        <button 
                            class="login-btn" 
                            :class="{ active: isLoginActive }" 
                            @click="setActive('login')"
                        >
                            Log In 
                        </button>
                        <button 
                            class="signup-btn" 
                            :class="{ active: !isLoginActive }" 
                            @click="setActive('signup')"
                        >
                            Sign Up
                        </button>
                    </div>
                </div>

                <transition :name="transitionName" mode="out-in">
                    <div v-if="isLoginActive" key="login" class="auth-slide">
                        <Auth />
                    </div>
                    <div v-else key="register" class="auth-slide">
                        <Register />
                    </div>
                </transition>
                <!-- Контейнер для форм (можно раскомментировать и настроить под ваши нужды) -->
                <!-- <div class="form-container">
                    <div class="form login-form" :class="{ active: isLoginActive }">
                        Login Form Content
                    </div>
                    <div class="form signup-form" :class="{ active: !isLoginActive }">
                        Signup Form Content
                    </div>
                </div> -->
            </div>
        </div>
    </div>
</template>

<script>
import auth from '~/components/auth.vue';
import register from './register.vue';


export default {
    data() {
        return {
            
            isLoginActive: false 
        };
    },
    methods: {
        setActive(type) {
            this.isLoginActive = type === 'login';
        }
    },
    watch: {
    isLoginActive(newVal, oldVal) {
      if (newVal) {
        // Переход к Login (Register -> Login)
        this.transitionName = 'slide-right';
      } else {
        // Переход к Register (Login -> Register)
        this.transitionName = 'slide-left';
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.signup-backround {
    background-image: url(public/signup.png);
    width: 100%;
    height: 100vh;
    background-position: center; /* Фиксирует изображение по центру */
    background-size: cover; /* Масштабирует изображение так, чтобы оно покрывало всю область, обрезая лишнее */
    background-repeat: no-repeat; 
}

.sign-up-txt {
    color: rgb(238, 238, 238);
    font-family: 'Inter', sans-serif;
    font-size: 32px;
    font-weight: 700;
    line-height: 130%;
    letter-spacing: -2%;
    text-align: left;
    width: 327px;
    height: 41px;
}

.sign-up-comment {
    width: 327px;
    height: 34px;
    color: rgb(255, 255, 255);
    font-family: 'Inter' ,sans-serif;
    font-size: 12px;
    font-weight: 400;
    line-height: 140%;
    letter-spacing: -1%;
    text-align: left;
}

.signup-block {
    width: 100%;
    height: 100%;
    border-radius: 16px 16px 0px 0px;
    background: #FFFFFF;
}

.signup-navigation {
    display: flex;
    justify-content: center;
    flex-direction: column;
}

.navigation {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.navigation {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.signup-navigation {
    text-align: center;
}


.signup-block {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.choose {
    position: relative;
    width: 327px;
    height: 36px;
    box-sizing: border-box;
    border: 2px solid rgb(245, 246, 249);
    border-radius: 7px;
    box-shadow: inset 0px 3px 6px 0px rgba(0, 0, 0, 0.02);
    background: rgb(245, 246, 249);
    display: flex;
    justify-content: space-between;
    overflow: hidden;
    margin: 24px 0px 0px 0px;
}

.slider {
    position: absolute;
    box-sizing: border-box;
    border-radius: 6px;
    width: 161px;
    height: 32px;
    background: rgb(255, 255, 255);
    transition: transform 0.3s ease; /* Плавная анимация */
    z-index: 1;
}

.slider.slide-right {
    transform: translateX(163px); /* Сдвиг вправо для Sign Up, учитываем отступы (161 + 2) */
}

.login-btn, .signup-btn {
    position: relative;
    width: 159px; /* Уменьшаем ширину кнопок на 2px с каждой стороны (161 - 2) */
    height: 32px;
    color: rgb(35, 36, 71);
    font-family: 'Inter', sans-serif;
    font-size: 14px;
    font-weight: 500;
    line-height: 0%;
    display: flex;
    justify-content: center;
    align-items: center;
    letter-spacing: -2%;
    box-sizing: border-box;
    border: none; /* Убираем границы кнопок */
    border-radius: 6px;
    background: transparent; /* Прозрачный фон кнопок */
    cursor: pointer;
    z-index: 2; /* Кнопки поверх ползунка */
    transition: color 0.3s ease; /* Плавное изменение цвета текста */
    margin:2px; /* Добавляем отступ 2px с каждой стороны кнопок */
    padding: 0px 0px 5px 0px;
}

/* Стили для активной кнопки */
.login-btn.active, .signup-btn.active {
    color: rgb(35, 36, 71);
    font-family: 'Inter', sans-serif;
    font-size: 14px;
    font-weight: 500;
    line-height: 0%;
    display: flex;
    justify-content: center;
    align-items: center;
    letter-spacing: -2%;
    text-align: center;
    padding: 0px 0px 5px 0px;
}

/* Стили для неактивной кнопки */
.login-btn:not(.active), .signup-btn:not(.active) {
    color: rgb(125, 125, 145);
    font-family: 'Inter', sans-serif;
    font-size: 14px;
    font-weight: 500;
    line-height: 0%;
    display: flex;
    justify-content: center;
    align-items: center;
    letter-spacing: -2%;
    text-align: center;
    padding: 0px 0px 5px 0px;
}

/* Стили для контейнера форм (раскомментируйте, если используете формы) */
/* .form-container {
    position: relative;
    width: 327px;
    height: 200px; 
    overflow: hidden;
    margin-top: 20px;
}

.form {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    transform: translateX(100%);
    transition: opacity 0.3s ease, transform 0.3s ease;
    background: white;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.1);
}

.form.active {
    opacity: 1;
    transform: translateX(0);
} */

.auth-container {
  position: relative;
  width: 300px; /* Примерная ширина контейнера */
  height: auto; /* Автоматическая высота */
  overflow: hidden; /* Чтобы скрыть выходящие элементы */
}

.auth-slide {
  top: 0; /* Удерживаем элемент вверху */
  left: 0;
  width: 100%;
}

/* Анимация при переходе влево (Login -> Register) */
.slide-left-enter-active,
.slide-left-leave-active {
  transition: transform 0.15s ease-in-out, opacity 0.15s ease-in-out; /* Анимируем только transform и opacity */
}

.slide-left-enter-from {
  transform: translateX(100%);
  opacity: 0;
}

.slide-left-enter-to {
  transform: translateX(0);
  opacity: 1;
}

.slide-left-leave-from {
  transform: translateX(0);
  opacity: 1;
}

.slide-left-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

/* Анимация при переходе вправо (Register -> Login) */
.slide-right-enter-active,
.slide-right-leave-active {
  transition: transform 0.15s ease-in-out, opacity 0.15s ease-in-out; /* Анимируем только transform и opacity */
}

.slide-right-enter-from {
  transform: translateX(-100%);
  opacity: 0;
}

.slide-right-enter-to {
  transform: translateX(0);
  opacity: 1;
}

.slide-right-leave-from {
  transform: translateX(0);
  opacity: 1;
}

.slide-right-leave-to {
  transform: translateX(100%);
  opacity: 0;
}
</style>