<template>
    <div>

      <Transition name="fade">
      <div v-if="isLoading" class="loading-container">
        <div id="app">
          <atom-spinner
            :animation-duration="1200"
            :size="60"
            :color="'#25A3E2'"
          />
        </div>
      </div>
    </Transition>

    <Transition name="fade-cont">
      <div class="backrounds">
          <form class="form-settings" id="registration-form" @submit.prevent="loginUser">
              <p class="form-placholders">Email</p>
              <input 
                type="email" 
                id="login-email" 
                name="login-email" 
                v-model="loginForm.email" 
                required
                class="form-blocks"
              />


              <p class="form-placholders-two">Password</p>
              <input 
                type="password" 
                id="login-password" 
                name="login-password" 
                v-model="loginForm.password" 
                required
                class="form-blocks"
              />
              <p class="errors-txt" v-if="showError">{{ catchError }}</p>

            <button class="register-btns" type="submit">Done</button>
          </form>
        </div>
    </Transition>
    </div>
</template>

<script>
import { AtomSpinner } from 'epic-spinners'
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = "https://dvdpezcwkklhlxafpyfl.supabase.co";
const supabaseKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR2ZHBlemN3a2tsaGx4YWZweWZsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAxNjE0MDcsImV4cCI6MjA1NTczNzQwN30.lhw8XGLjw2GBhSLwuUaMGlz67vt9k1MztLUnRE7qBGM";
const supabase = createClient(supabaseUrl, supabaseKey);

export default {
  data() {
    return {
      showError: false,
      catchError: '',
      isLoading: true,
      registrationForm: {
        name: '',
        email: '',
        password: ''
      },
      loginForm: {
        email: '',
        password: ''
      },
      registrationSuccess: false,
      registrationError: null,
      loginError: null,
      user: null,
      userName: null,
      userBalance: 0 // Добавляем свойство для хранения баланса пользователя
    };
    
  },
  mounted() {
    this.checkSession();
  },
  methods: {
    async checkSession() {
      try {
        const { data: { session }, error: sessionError } = await supabase.auth.getSession();

        if (sessionError) {
          console.error("Ошибка при получении сессии:", sessionError);
          return;
        }

        if (session) {
          await this.fetchUser();
        } else {
          this.user = null;
          this.userName = null;
          this.userBalance = 0; // Сбрасываем баланс при отсутствии пользователя
        }
      } catch (error) {
        console.error("Общая ошибка при проверке сессии:", error);
      } finally {
        this.isLoading = false;
      }
    },

    async fetchUser() {
      try {
        const { data: { user }, error: userError } = await supabase.auth.getUser();

        if (userError) {
          console.error("Ошибка при получении данных пользователя:", userError);
          return;
        }

        if (user) {
          this.user = user;
          this.userName = user.user_metadata.full_name || user.email;
          await this.fetchUserProfile(user.id); // Получаем профиль пользователя и баланс
        }
      } catch (error) {
        console.error("Общая ошибка при получении пользователя:", error);
      }
    },

    async fetchUserProfile(userId) {
      try {
        const { data, error } = await supabase
          .from('user_profiles') // Замените 'user_profiles' на имя вашей таблицы профилей
          .select('balance')
          .eq('id', userId) // Фильтруем по user.id (из auth.users)
          .single();

        if (error) {
          console.error("Ошибка при получении профиля пользователя:", error);
          this.userBalance = 'Ошибка загрузки'; // Обработка ошибки баланса
        } else if (data) {
          this.userBalance = data.balance || 0; // Используем полученный баланс или 0 по умолчанию
        } else {
          this.userBalance = 0; // Профиль не найден, устанавливаем баланс в 0
        }
      } catch (error) {
        console.error("Общая ошибка при получении профиля пользователя:", error);
        this.userBalance = 'Ошибка загрузки'; // Общая ошибка при загрузке баланса
      }
    },

    async loginUser() {
      const router = useRouter()
      this.loginError = null;
      try {
        const { error, data } = await supabase.auth.signInWithPassword({
          email: this.loginForm.email,
          password: this.loginForm.password,
        });

        if (error) {
          this.catchError = 'Incorrect password or email'
          this.triggerError();
          console.error('Ошибка авторизации:', error);
          this.loginError = error.message;
        } else {
          console.log('Авторизация успешна:', data);
          this.loginForm = { email: '', password: '' };
          await this.fetchUser();
          await this.checkSession();
          router.push({ path: "/main" })
        }
      } catch (error) {
        console.error('Общая ошибка при авторизации:', error);
        this.loginError = 'Произошла ошибка при авторизации. Пожалуйста, попробуйте позже.';
      } finally {
        this.isLoading = false;
      }
    },

    triggerError() {
      this.showError = true;
      setTimeout(() => {
        this.showError = false;
      }, 2000);
    },

    async logoutUser() {
      try {
        const { error } = await supabase.auth.signOut();
        if (error) {
          console.error('Ошибка выхода из системы:', error);
        } else {
          console.log('Выход из системы успешен');
          this.user = null;
          this.userName = null;
          this.userBalance = 0; // Сбрасываем баланс при выходе
        }
      } catch (error) {
        console.error('Общая ошибка при выходе из системы:', error);
      } finally {
        this.isLoading = false;
        await this.checkSession();
      } 
    }
  }
};
</script>

<style>
* {
  margin: 0px;
  padding: 0px;
}

html {
  overflow: hidden;
  touch-action: manipulation; /* Отключает двойной тап и зум */
}

ul li {
  list-style: none;
  float: left;
}

button {
  background: none;
  border: none;
}

.backrounds {
    background: none;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
}

.register-form {
  border-radius: 32px;
  width: 335px;
  height: 415px;
  background: rgb(23, 24, 28);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 100px 0px 0px 0px;
}

.form-placholders {
  color: rgb(108, 114, 120);
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 400;
  line-height: 160%;
  letter-spacing: -2%;
  text-align: left;
  width: 327px;
  height: 21px;
  margin: 25px 0px 2px 0px;
}

.form-placholders-two {
  color: rgb(108, 114, 120);
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 400;
  line-height: 160%;
  letter-spacing: -2%;
  text-align: left;
  width: 327px;
  height: 21px;
  margin: 15px 0px 2px 0px;
}

.form-blocks {
  box-sizing: border-box;
  border: 1px solid rgb(237, 241, 243);
  border-radius: 10px;
  width: 327px;
  height: 46px;
  box-shadow: 0px 1px 2px 0px rgba(228, 229, 231, 0.24);
  background: rgb(255, 255, 255);
  color: rgb(26, 28, 30);
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 140%;
  letter-spacing: -1%;
  text-align: left;
}

input {
  outline:none;
  padding-top:5px;
  padding-bottom:5px;
  padding-right: 12px;
  padding-left: 12px;
}

::placeholder {
  color: white;
  opacity: 0.7;
}

.register-txt {
  color: rgb(255, 255, 255);
  font-family: 'Offside', sans-serif;
  font-size: 36px;
  font-weight: 400;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: center;
  width: 227px;
  height: 40px;
  margin: 0px 0px 19px 0px;
}

.register-btns {
  box-sizing: border-box;
  border-radius: 10px;
  color: rgb(255, 255, 255);
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 140%;
  letter-spacing: -1%;
  text-align: center;
  width: 327px;
  height: 48px;
  box-shadow: 0px 0px 0px 1px rgb(55, 93, 251),0px 1px 2px 0px rgba(37, 62, 167, 0.48);
  background: linear-gradient(180.00deg, rgba(255, 255, 255, 0.12),rgba(255, 255, 255, 0) 100%),rgb(29, 97, 231);
  margin: 24px 0px 0px 0px;
}

.form-settings {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.auth-txt {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0%;
  text-align: center;
  width: 295px;
  height: 17px;
  margin: 4px 0px 0px 0px;
}

.html {
  touch-action: manipulation; /* Отключает двойной тап и зум */
} 

.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

.fade-enter-to,
.fade-leave-from {
    opacity: 1;
}

/* Анимация для основного контента (fade-cont) */
.fade-cont-enter-active {
    transition: opacity 0s ease;
}

.fade-cont-leave-active {
    transition: opacity 0s ease;
}

.fade-cont-enter-from,
.fade-cont-leave-to {
    opacity: 0;
}

.fade-cont-enter-to,
.fade-cont-leave-from {
    opacity: 1;
}

.errors-txt {
  color: rgb(244, 82, 82);
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  font-weight: 600;
  line-height: 14px;
  letter-spacing: 0%;
  text-align: center;
  height: 14px;
  margin: 10px 0px 0px 0px ;
}
</style>