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
        <div v-if="requestSuccess" class="modal">
          <div class="modal-content">
            <div>
              <img class="approve-gif" src="public/approve.gif">
            </div>
            <p class="approve-txt">Registration was successful</p>
          </div>
        </div>
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
      requestSuccess: false,
      requestError: null,
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
          console.error("Error getting session:", sessionError);
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
        console.error("General error when checking session:", error);
      }
    },

    async fetchUser() {
      try {
        const { data: { user }, error: userError } = await supabase.auth.getUser();

        if (userError) {
          console.error("Error retrieving user data:", userError);
          return;
        }

        if (user) {
          this.user = user;
          this.userName = user.user_metadata.full_name || user.email;
          this.userEmail = user.email;
          await this.fetchUserProfile(user.id);
        }
      } catch (error) {
        console.error("General error while getting user:", error);
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
          console.error("Error getting user profile:", error);
          this.userBalance = 'Loading error';
        } else if (data) {
          this.userBalance = data.balance || 0;
          this.referredBy = data.referred_by || ''; // Сохраняем реферальный код
        } else {
          this.userBalance = 0;
          this.referredBy = ''; // Сохраняем реферальный код
        }
      } catch (error) {
        console.error("General error when retrieving user profile:", error);
        this.userBalance = 'Loading error';
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
            console.error('Registration error:', error);
            this.registrationError = error.message;
          } else {
            console.log('Registration successful:', data);
            this.registrationSuccess = true;
            this.requestSuccess = true; // Устанавливаем сразу после успешной регистрации

            // Создаем профиль пользователя с реферальным кодом
            await this.createUserProfile(data.user.id, this.registrationForm.name);

            this.registrationForm = { name: '', email: '', password: '' };
            this.requestForm.productName = ''; // Очищаем поле формы
            
            // Перенаправляем на страницу входа
            //window.location.reload();
            setTimeout(() => {
              this.requestSuccess = false; // Закрываем модальное окно через 5 секунд
            }, 50000);

            //router.push({ path: "/Tonlog" })
          }
        } catch (error) {
          console.error('General registration error:', error);
          this.registrationError = 'There was an error registering. Please try again later.';
          setTimeout(() => {
            this.requestSuccess = false; // Закрываем модальное окно через 5 секунд в случае ошибки
          }, 5000);
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
          console.error("Error creating user profile:", error);
          alert('Registration was successful, but an error occurred while creating your profile. Please contact support.');
        } else {
          console.log("User profile created successfully");

          // Обрабатываем реферальный код, если он был в URL
          if (this.referrerCode) {
            await this.incrementReferralCount(this.referrerCode);
          }
        }
      } catch (error) {
        console.error("General error when creating a user profile:", error);
        alert('Registration was successful, but an error occurred while creating your profile. Please contact support.');
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
          console.error("Error searching for referrer:", fetchError);
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
            console.error("Error updating referral counter:", updateError);
          } else {
            console.log("Referral counter successfully updated");
          }
        }
      } catch (error) {
        console.error("General error when updating referral counter:", error);
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

html {
  overflow: hidden;
  touch-action: manipulation; /* Отключает двойной тап и зум */
}

button {
background: none;
border: none;
}

.backroundd {
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

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  animation: fadeIn 0.5s;
  z-index: 10;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  text-align: center;
  animation: zoomIn 0.5s;
}

.checkmark {
  width: 50px;
  height: 50px;
  border-radius: 15px;
  background-color: #4CAF50;
  display: inline-block;
  margin-bottom: 10px;
  animation: pulse 1s infinite;
}

.checkmark svg {
  width: 100%;
  height: 100%;
  fill: #fff;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes zoomIn {
  from {
    transform: scale(0.5);
  }
  to {
    transform: scale(1);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.approve-txt {
  background: linear-gradient(153.43deg, rgb(29, 97, 231),rgb(61, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 600;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: center;
}

.approve-gif {
   height: 188px;
   width: 188px;
}
</style>