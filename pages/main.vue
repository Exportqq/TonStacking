<template>
  <Head>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, user-scalable=no">
  </Head>
    <div>
      <div class="backround">
        <div class="header-navigation">
          <div class="header-block">
            <ul style="display: flex; align-items: center; margin-top: 15px;">
              <li>
                <img src="public/user.svg">
              </li>
              <li>
                <p class="user-name">{{ userName }}</p>
              </li>
              <li>
                <button @click="logoutUser"><img class="log-out-img" src="public/logout.svg"></button>
              </li>
            </ul>
          </div>
        </div>

        <div>
          <p class="balance">{{ userBalance }} TON</p>
          <div class="transaction-navigation">
            <button class="transaction-btn">
              <img src="public/send.svg">
              <p class="transaction-txt">Send</p>
            </button>
            <div class="vertical-line"></div>
            <button class="transaction-btn">
              <img src="public/wallet.svg">
              <p class="transaction-txt">Receive</p>
            </button>
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
      const router = useRouter()
      try {
        const { error } = await supabase.auth.signOut();
        if (error) {
          console.error('Ошибка выхода из системы:', error);
        } else {
          console.log('Выход из системы успешен');
          this.user = null;
          this.userName = null;
          this.userBalance = 0; // Сбрасываем баланс при выходе
          router.push({ path: "/login" })
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
html {
  overflow: hidden;
  touch-action: manipulation; /* Отключает двойной тап и зум */
}

* {
  margin: 0px;
  padding: 0px;
}

.backround {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  width: 100%;
  height: 100%;
  background: linear-gradient(180.00deg, rgb(14, 23, 88),rgb(12, 13, 14) 20.312%),rgb(12, 13, 14);
}

ul li {
  list-style: none;
  float: left;
}

button {
  background: none;
  border: none;
}

.user-name {
  color: rgb(255, 255, 255);
  font-family: 'Outfit', sans-serif;
  font-size: 20px;
  font-weight: 500;
  line-height: 25px;
  letter-spacing: 0px;
  text-align: left;
  width: 54px;
  height: 30px;
  margin: 0px 0px 0px 8px;
}

.log-out-img {
  width: 24px;
  height: 24px;
  margin: 0px 0px 0px 228px;
}

.header-block {
  width: 345px;
}

.header-navigation {
  display: flex;
  justify-content: center;
}

.balance {
  color: rgb(255, 255, 255);
  font-family: 'Outfit', sans-serif;
  font-size: 48px;
  font-weight: 500;
  line-height: 60px;
  letter-spacing: 0px;
  text-align: center;
  width: 100%;
  height: 60px;
  margin: 59px 0px 0px 0px;
}

.transaction-txt {
  width: 100%;
  height: 20px;
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  font-weight: 500;
  line-height: 20px;
  letter-spacing: -1%;
  text-align: center;
  margin: 6px 0px 0px 0px;
}

.transaction-btn {
  width: 85px;
  height: 76px;
  border-radius: 24px;
  background: rgb(23, 24, 28);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  float: left;
}

.vertical-line {
  width: 1px;
  background: rgb(23, 24, 28);
  height: 50px;
  float: left;
  margin: 0px 20px 0px 20px;
}

.transaction-navigation {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 29px 0px 0px 0px;
}

</style>