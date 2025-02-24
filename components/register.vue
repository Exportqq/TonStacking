<template>
    <div>
        <div class="backround">
          <form class="form-settings" id="registration-form" @submit.prevent="registerUser">
            <div class="register-form">
              <p class="register-txt">Registration</p>

              <p class="form-placholder">Name</p>
              <input 
                type="text" 
                id="reg-name" 
                name="reg-name" 
                v-model="registrationForm.name" 
                required
                class="form-block"
              />
          
              <p class="form-placholder">Password</p>
              <input 
                type="password" 
                id="reg-password" 
                name="reg-password" 
                v-model="registrationForm.password" 
                required
                class="form-block"
              />

              <p class="form-placholder">Email</p>
              <input 
                type="email" 
                id="reg-email" 
                name="reg-email" 
                v-model="registrationForm.email" 
                required
                class="form-block"
              />
            </div>

            <button class="register-btn" type="submit">Done</button>
          </form>
        </div>
    </div>
</template>

<script>
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = "https://dvdpezcwkklhlxafpyfl.supabase.co";
const supabaseKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR2ZHBlemN3a2tsaGx4YWZweWZsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAxNjE0MDcsImV4cCI6MjA1NTczNzQwN30.lhw8XGLjw2GBhSLwuUaMGlz67vt9k1MztLUnRE7qBGM";
const supabase = createClient(supabaseUrl, supabaseKey);

export default {
  data() {
    return {
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


    async registerUser() {
      this.registrationError = null;
      this.registrationSuccess = false;

      try {
        const { error, data } = await supabase.auth.signUp({
          email: this.registrationForm.email,
          password: this.registrationForm.password,
          options: {
            data: {
              full_name: this.registrationForm.name,
            }
          }
        });

        if (error) {
          console.error('Ошибка регистрации:', error);
          this.registrationError = error.message;
        } else {
          console.log('Регистрация успешна:', data);
          this.registrationSuccess = true;
          this.registrationForm = { name: '', email: '', password: '' };

          // Создаем профиль пользователя в 'user_profiles' таблице после успешной регистрации
          await this.createUserProfile(data.user.id, this.registrationForm.name);
        }
      } catch (error) {
        console.error('Общая ошибка при регистрации:', error);
        this.registrationError = 'Произошла ошибка при регистрации. Пожалуйста, попробуйте позже.';
      }
    },

    async createUserProfile(userId, userName) {
      try {
        const { error } = await supabase
          .from('user_profiles') // Замените 'user_profiles' на имя вашей таблицы профилей
          .insert([{ id: userId, full_name: userName, balance: 0 }]); // Имя можно сохранить и в профиле, если нужно

        if (error) {
          console.error("Ошибка при создании профиля пользователя:", error);
          // Регистрация пользователя в Auth успешна, но создание профиля не удалось.
          // Вам нужно решить, как обрабатывать такую ситуацию.
          // Например, можно удалить пользователя из Auth или показать сообщение об ошибке и попросить пользователя связаться с поддержкой.
          alert('Регистрация прошла успешно, но возникла ошибка при создании профиля. Обратитесь в поддержку.');
        } else {
          console.log("Профиль пользователя создан успешно");
        }
      } catch (error) {
        console.error("Общая ошибка при создании профиля пользователя:", error);
        alert('Регистрация прошла успешно, но возникла ошибка при создании профиля. Обратитесь в поддержку.');
      }
    },

    async loginUser() {
      this.loginError = null;
      try {
        const { error, data } = await supabase.auth.signInWithPassword({
          email: this.loginForm.email,
          password: this.loginForm.password,
        });

        if (error) {
          console.error('Ошибка авторизации:', error);
          this.loginError = error.message;
        } else {
          console.log('Авторизация успешна:', data);
          this.loginForm = { email: '', password: '' };
          await this.fetchUser();
          await this.checkSession();
        }
      } catch (error) {
        console.error('Общая ошибка при авторизации:', error);
        this.loginError = 'Произошла ошибка при авторизации. Пожалуйста, попробуйте позже.';
      }
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

ul li {
  list-style: none;
  float: left;
}

button {
  background: none;
  border: none;
}

.backround {
    background: rgb(12, 13, 14);
    width: 100%;
    height: 1000px;
    display: flex;
    justify-content: center;
    align-items: center;
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
}

.form-placholder {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: left;
  height: 21px;
  width: 284px;
  margin: 0px 0px 6px 0px;
}

.form-block {
  border-radius: 5px;
  background: rgb(32, 66, 245);
  width: 260px;
  height: 30px;
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: left;
  border: none;
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
  margin: 0px 0px 44px 0px;
}

.register-btn {
  border-radius: 12px;
  width: 295px;
  height: 52px;
  background: rgb(10, 40, 244);
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 600;
  line-height: 24px;
  letter-spacing: -1%;
  text-align: center;
  margin: 15px 0px 0px 0px;
}

.form-settings {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>