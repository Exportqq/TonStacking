<template>
  <div>
    <!-- Добавленная секция TON Connect -->
    <div class="ton-connect-section">
        <h2>TON Кошелек</h2>
        <div v-if="account">
          <p>Адрес: {{ shortAddress(account.address) }}</p>
          <p>Баланс: {{ tonBalance }} TON</p>
          <button @click="disconnectWallet">Отключить кошелек</button>
        </div>
        <button v-else @click="connectWallet">Подключить TON кошелек</button>
      </div>

      <button @click="logoutUser">Выйти</button>
    <div v-if="user">
      <h1>Добро пожаловать, {{ userName }}! Баланс: {{ userBalance }}</h1>
      <button @click="logoutUser">Выйти</button>
    </div>
    <div v-else>
      <h1>Регистрация</h1>
      <form id="registration-form" @submit.prevent="registerUser">
        <div>
          <label for="reg-name">Имя:</label>
          <input type="text" id="reg-name" name="reg-name" v-model="registrationForm.name" required>
        </div>
        <div>
          <label for="reg-email">Email:</label>
          <input type="email" id="reg-email" name="reg-email" v-model="registrationForm.email" required>
        </div>
        <div>
          <label for="reg-password">Пароль:</label>
          <input type="password" id="reg-password" name="reg-password" v-model="registrationForm.password" required>
        </div>
        <button type="submit">Зарегистрироваться</button>
      </form>
      <div v-if="registrationSuccess">
        Регистрация прошла успешно!
      </div>
      <div v-if="registrationError" class="error-message">
        Ошибка регистрации: {{ registrationError }}
      </div>

      <hr>

      <h1>Авторизация</h1>
      <form id="login-form" @submit.prevent="loginUser">
        <div>
          <label for="login-email">Email:</label>
          <input type="email" id="login-email" name="login-email" v-model="loginForm.email" required>
        </div>
        <div>
          <label for="login-password">Пароль:</label>
          <input type="password" id="login-password" name="login-password" v-model="loginForm.password" required>
        </div>
        <button type="submit">Войти</button>
      </form>
      <div v-if="loginError" class="error-message">
        Ошибка авторизации: {{ loginError }}
      </div>
    </div>
  </div>
</template>

<script>
import { createClient } from '@supabase/supabase-js';
import { TonConnectUI } from '@tonconnect/ui';

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
      tonConnectUI: null,
      account: null,
      tonBalance: '0.00',
      userBalance: 0 // Добавляем свойство для хранения баланса пользователя
    };
  },
  mounted() {
    this.checkSession();
    this.checkSession();
    this.initTonConnect();
  },
  methods: {

    async initTonConnect() {
      this.tonConnectUI = new TonConnectUI({
        manifestUrl: 'https://your-domain.com/tonconnect-manifest.json',
        buttonRootId: 'ton-connect-button'
      });

      this.tonConnectUI.onStatusChange(wallet => {
        this.account = wallet?.account;
        if (wallet?.account) {
          this.fetchTonBalance(wallet.account.address);
        } else {
          this.tonBalance = '0.00';
        }
      });
    },

    shortAddress(address) {
      return address ? `${address.slice(0, 4)}...${address.slice(-4)}` : '';
    },

    connectWallet() {
      this.tonConnectUI.connect();
    },

    disconnectWallet() {
      this.tonConnectUI.disconnect();
    },

    async fetchTonBalance(address) {
      try {
        const response = await fetch(`https://tonapi.io/v2/accounts/${address}`);
        const data = await response.json();
        this.tonBalance = (data.balance / 1000000000).toFixed(2);
      } catch (error) {
        console.error('Ошибка получения баланса:', error);
        this.tonBalance = 'Ошибка';
      }
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

<style scoped>
/* Стили из предыдущего примера */
#registration-form, #login-form {
  display: flex;
  flex-direction: column;
  max-width: 300px;
  margin-top: 20px;
}

#registration-form div, #login-form div {
  margin-bottom: 10px;
}

#registration-form label, #login-form label {
  display: block;
  margin-bottom: 5px;
}

#registration-form input, #login-form input,
#registration-form button, #login-form button {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  width: 100%;
}

#registration-form button, #login-form button, button {
  background-color: #4CAF50;
  color: white;
  cursor: pointer;
  margin-top: 10px;
}

#registration-form button:hover, #login-form button:hover, button:hover {
  background-color: #45a049;
}

.error-message {
  color: red;
  margin-top: 10px;
}

/* Добавленные стили для TON Connect */
.ton-connect-section {
  margin: 20px 0;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.ton-connect-section h2 {
  margin-bottom: 10px;
  color: #333;
}

.ton-connect-section button {
  background-color: #0088cc;
  margin: 5px 0;
}

.ton-connect-section button:hover {
  background-color: #0077b3;
}
</style>