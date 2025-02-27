<template>
  <div>
      <div class="backroundd">
        <form class="form-settings" id="registration-form" @submit.prevent="registerUser">
            <p class="form-placholders">Email</p>
            <input 
              type="email" 
              id="reg-email" 
              name="reg-email" 
              v-model="registrationForm.email" 
              required
              class="form-blocks"
            />
        
            <p class="form-placholders-two">Name</p>
            <input 
              type="text" 
              id="reg-name" 
              name="reg-name" 
              v-model="registrationForm.name"
              required
              class="form-blocks"
            />

            <p class="form-placholders-two">Password</p>
            <input 
              type="password" 
              id="reg-password" 
              name="reg-password" 
              v-model="registrationForm.password"
              required
              class="form-blocks"
            />

            <p class="errors-txt" v-if="showError">{{ catchError }}</p>

            <!-- Показываем информацию о реферале, если он есть -->
            <p v-if="referrerCode" class="referral-info">
              Приглашены пользователем с кодом: {{ referrerCode }}
            </p>

          <button class="register-btns" type="submit">Done</button>
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
      catchError: '',
      showError: false,
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
      userBalance: 0,
      referrerCode: '' // Поле для реферального кода
    };
  },

  mounted() {
    this.checkSession();

    // Получаем код реферера из URL, если он есть
    const urlParams = new URLSearchParams(window.location.search);
    this.referrerCode = urlParams.get('ref') || '';
  },
  methods: {
    triggerError() {
      this.showError = true;
      setTimeout(() => {
        this.showError = false;
      }, 2000);
    },
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
          this.userBalance = 0;
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
          this.userEmail = user.email;
          await this.fetchUserProfile(user.id);
        }
      } catch (error) {
        console.error("Общая ошибка при получении пользователя:", error);
      }
    },

    async fetchUserProfile(userId) {
      try {
        const { data, error } = await supabase
          .from('user_profiles')
          .select('balance, referred_by')
          .eq('id', userId)
          .single();

        if (error) {
          console.error("Ошибка при получении профиля пользователя:", error);
          this.userBalance = 'Ошибка загрузки';
        } else if (data) {
          this.userBalance = data.balance || 0;
          this.referredBy = data.referred_by || ''; // Сохраняем реферальный код
        } else {
          this.userBalance = 0;
          this.referredBy = ''; // Сохраняем реферальный код
        }
      } catch (error) {
        console.error("Общая ошибка при получении профиля пользователя:", error);
        this.userBalance = 'Ошибка загрузки';
        this.referredBy = ''; // Сохраняем реферальный код
      }
    },

    // Генерация уникального реферального кода
    generateReferralCode() {
      return Math.random().toString(36).substring(2, 10); // Генерируем случайный код из 8 символов
    },

    async registerUser() {
      const router = useRouter()
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
          this.catchError = 'Password is not less than 6 characters or this email is busy'
          this.triggerError();
          console.error('Ошибка регистрации:', error);
          this.registrationError = error.message;
        } else {
          console.log('Регистрация успешна:', data);
          this.registrationSuccess = true;

          // Создаем профиль пользователя с реферальным кодом
          await this.createUserProfile(data.user.id, this.registrationForm.name);

          this.registrationForm = { name: '', email: '', password: '' };

          // Перенаправляем на страницу входа
          router.push({ path: "/login" })
        }
      } catch (error) {
        console.error('Общая ошибка при регистрации:', error);
        this.registrationError = 'Произошла ошибка при регистрации. Пожалуйста, попробуйте позже.';
      }
    },

    async createUserProfile(userId, userName) {
      try {
        // Генерируем реферальный код для нового пользователя
        const referralCode = this.generateReferralCode();

        const { error } = await supabase
          .from('user_profiles')
          .insert([{
            id: userId,
            full_name: userName,
            balance: 0,
            referral_code: referralCode,
            referral_count: 0,
            referred_by: this.referrerCode // Сохраняем реферальный код
          }]);

        if (error) {
          console.error("Ошибка при создании профиля пользователя:", error);
          alert('Регистрация прошла успешно, но возникла ошибка при создании профиля. Обратитесь в поддержку.');
        } else {
          console.log("Профиль пользователя создан успешно");

          // Обрабатываем реферальный код, если он был в URL
          if (this.referrerCode) {
            await this.incrementReferralCount(this.referrerCode);
          }
        }
      } catch (error) {
        console.error("Общая ошибка при создании профиля пользователя:", error);
        alert('Регистрация прошла успешно, но возникла ошибка при создании профиля. Обратитесь в поддержку.');
      }
    },

    // Увеличиваем счетчик рефералов у пригласившего пользователя
    async incrementReferralCount(referralCode) {
      try {
        // Находим пользователя по реферальному коду
        const { data: referrer, error: fetchError } = await supabase
          .from('user_profiles')
          .select('id, referral_count')
          .eq('referral_code', referralCode)
          .single();

        if (fetchError) {
          console.error("Ошибка при поиске реферера:", fetchError);
          return;
        }

        if (referrer) {
          // Увеличиваем счетчик рефералов на 1
          const newCount = (referrer.referral_count || 0) + 1;

          const { error: updateError } = await supabase
            .from('user_profiles')
            .update({ referral_count: newCount })
            .eq('id', referrer.id);

          if (updateError) {
            console.error("Ошибка при обновлении счетчика рефералов:", updateError);
          } else {
            console.log("Счетчик рефералов успешно обновлен");
          }
        }
      } catch (error) {
        console.error("Общая ошибка при обновлении счетчика рефералов:", error);
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

.backroundd {
  background: none;
  width: 100%;
  height: 1200px;
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
margin: 25px 0px 6px 0px;
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
margin: 0px 0px 19px 0px;
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

html {
touch-action: manipulation; /* Отключает двойной тап и зум */
}

/* Стиль для отображения информации о реферале */
.referral-info {
color: rgb(109, 188, 53);
font-family: 'Montserrat', sans-serif;
font-size: 12px;
font-weight: 500;
margin-top: 15px;
text-align: center;
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
  width: 240px;
}
</style>