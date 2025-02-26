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

        <!-- Добавлен блок для реферальной системы -->
        <div class="referral-block">
          <p class="referral-title">Invite friends</p>
          <p class="referral-count">Friends invited: {{ referralCount }}</p>
          <div class="referral-link-container">
            <input type="text" class="referral-link" :value="referralLink" readonly />
            <button @click="copyReferralLink" class="copy-btn">Copy</button>
          </div>
        </div>

        <div class="staking-header">
          <div class="horizontal-line"></div>
          <p class="stake-txt">Staking</p>
          <div class="horizontal-line-two"></div>
        </div>
        <div style="display: flex;flex-direction: column;align-items: center; overflow: scroll;">
          <div class="stake-block">
            <div style="float: left;">
              <p class="stake-block-txt">Staking for 21 days</p>
              <NuxtLink to="lowTON">
                <button class="stake-block-btn">Stake</button>
              </NuxtLink>
            </div>
            <div style="float: left;">
              <div style="display: flex;flex-direction: column;align-items: center;">
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">180</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
                <ul style="margin: 3px 24px 0px 0px;">
                  <li>
                    <p class="ton-procent">36%</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg">
                  </li>
                </ul>
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">245</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
              </div>
            </div>
          </div>

          <div class="stake-block">
            <div style="float: left;">
              <p class="stake-block-txt">Staking for 21 days</p>
              <button class="stake-block-btn">Stake</button>
            </div>
            <div style="float: left;">
              <div style="display: flex;flex-direction: column;align-items: center;">
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">250</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
                <ul style="margin: 3px 24px 0px 0px;">
                  <li>
                    <p class="ton-procent">44%</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg">
                  </li>
                </ul>
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">360</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
              </div>
            </div>
          </div>

          <div class="stake-block">
            <div style="float: left;">
              <p class="stake-block-txt">Staking for 14 days</p>
              <button class="stake-block-btn">Stake</button>
            </div>
            <div style="float: left;">
              <div style="display: flex;flex-direction: column;align-items: center;">
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">390</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
                <ul style="margin: 3px 24px 0px 0px;">
                  <li>
                    <p class="ton-procent">51%</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg">
                  </li>
                </ul>
                <ul class="ton-list">
                  <li>
                    <p class="ton-price">590</p>
                  </li>
                  <li>
                    <img style="margin: 0px 0px 0px 4px;" src="public/path.svg">
                  </li>
                </ul>
              </div>
            </div>
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
      userBalance: 0,
      referralCount: 0,
      referralCode: '',
      referralLink: '',
      referrerCode: '' // Для хранения кода реферера при регистрации
    };
  },
  name: 'register',
  async asyncData({ query }) {
    const ref = query.ref
    // Обработка параметра ref
  },
  mounted() {
    this.checkSession();
    // Получаем код реферера из URL, если он есть
    const urlParams = new URLSearchParams(window.location.search);
    this.referrerCode = urlParams.get('ref') || '';
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
          this.userBalance = 0;
          this.referralCount = 0;
          this.referralCode = '';
          this.referralLink = '';
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
          .select('balance, referral_code, referral_count')
          .eq('id', userId)
          .single();

        if (error) {
          console.error("Ошибка при получении профиля пользователя:", error);
          this.userBalance = 'Ошибка загрузки';
        } else if (data) {
          this.userBalance = data.balance || 0;
          this.referralCode = data.referral_code || '';
          this.referralCount = data.referral_count || 0;
          
          // Создаем реферальную ссылку - ИЗМЕНЕНО с register на registration
          const baseUrl = window.location.origin;
          this.referralLink = `${baseUrl}/?ref=${this.referralCode}`;
        } else {
          this.userBalance = 0;
          this.referralCount = 0;
          this.referralLink = '';
        }
      } catch (error) {
        console.error("Общая ошибка при получении профиля пользователя:", error);
        this.userBalance = 'Ошибка загрузки';
      }
    },

    // Метод для копирования реферальной ссылки в буфер обмена
    async copyReferralLink() {
      try {
        await navigator.clipboard.writeText(this.referralLink);
        alert('Реферальная ссылка скопирована!');
      } catch (err) {
        console.error('Не удалось скопировать ссылку: ', err);
        alert('Не удалось скопировать ссылку. Попробуйте скопировать вручную.');
      }
    },

    // Генерация уникального реферального кода
    generateReferralCode() {
      return Math.random().toString(36).substring(2, 10); // Генерируем случайный код из 8 символов
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
          
          // Создаем профиль пользователя с реферальным кодом
          await this.createUserProfile(data.user.id, this.registrationForm.name);
          
          // Обрабатываем реферальный код, если он был в URL
          if (this.referrerCode) {
            await this.incrementReferralCount(this.referrerCode);
          }
          
          this.registrationForm = { name: '', email: '', password: '' };
        }
      } catch (error) {
        console.error('Общая ошибка при регистрации:', error);
        this.registrationError = 'Произошла ошибка при регистрации. Пожалуйста, попробуйте позже.';
      }
    },

    async createUserProfile(userId, userName) {
      try {
        const referralCode = this.generateReferralCode();
        
        const { error } = await supabase
          .from('user_profiles')
          .insert([{ 
            id: userId, 
            full_name: userName, 
            balance: 0,
            referral_code: referralCode,
            referral_count: 0
          }]);

        if (error) {
          console.error("Ошибка при создании профиля пользователя:", error);
          alert('Регистрация прошла успешно, но возникла ошибка при создании профиля. Обратитесь в поддержку.');
        } else {
          console.log("Профиль пользователя создан успешно");
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
          this.userBalance = 0;
          this.referralCount = 0;
          this.referralCode = '';
          this.referralLink = '';
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
  align-items: center;
  flex-direction: column;
  justify-content: flex-start;
  width: 100%;
  height: 100vh;
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

/* Стили для реферальной системы */
.referral-block {
  width: 339px;
  background: rgb(32, 34, 43);
  border-radius: 16px;
  padding: 15px;
  margin-top: 20px;
}

.referral-title {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 18px;
  font-weight: 500;
  margin-bottom: 10px;
  text-align: center;
}

.referral-count {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 14px;
  margin-bottom: 10px;
}

.referral-link-container {
  display: flex;
  align-items: center;
  margin-bottom: 5px;
}

.referral-link {
  flex: 1;
  background: rgb(23, 24, 28);
  border: none;
  border-radius: 4px;
  padding: 8px;
  color: white;
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  margin-right: 5px;
}

.copy-btn {
  background: linear-gradient(120.09deg, rgb(0, 33, 255) -33.497%,rgba(0, 33, 255, 0) 281.515%,rgb(10, 40, 244) 281.515%,rgb(10, 40, 244) 281.515%);
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  cursor: pointer;
}

.staking-header {
  display: flex;
  align-items: center;
  width: 100%;
  margin: 0;
  padding: 0;
}

.staking-header {
  display: flex;
  align-items: center; /* Выравнивание элементов по центру по вертикали */
  width: 100%;
  margin: 0;
  padding: 0;
  margin: 30px 0px 20px 0px;
}

.horizontal-line {
  flex-grow: 1; /* Линии занимают все доступное пространство */
  height: 1px;
  background: linear-gradient(90deg, rgb(21, 53, 244), rgba(14, 21, 74, 0) 100%);
}

.horizontal-line-two {
  transform: rotate(180deg);
  flex-grow: 1; /* Линии занимают все доступное пространство */
  height: 1px;
  background: linear-gradient(90deg, rgb(21, 53, 244), rgba(14, 21, 74, 0) 100%);
}

.horizontal-line:last-child {
  /* Разворачиваем градиент для правой линии */
  background: linear-gradient(270deg, rgb(21, 53, 244), rgba(14, 21, 74, 0) 100%);
}

.stake-txt {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 24px;
  font-weight: 400;
  margin: 0 20px; /* Отступы от текста до линий */
  padding: 0;
  line-height: 1; /* Уменьшаем line-height для лучшего выравнивания */
}

.stake-block {
  border-radius: 16px;
  width: 339px;
  height: 100px;
  background: rgb(32, 34, 43);
  display: flex;
  align-items: center;
  margin: 15px 0px 0px 0px;
}

.stake-block-txt {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: left;
  width: 244px;
  height: 21px;
  margin: 0px 0px 0px 14px;
}

.stake-block-btn {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: center;
  width: 103px;
  height: 24px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 37px 0px 0px 14px;
  background: linear-gradient(120.09deg, rgb(0, 33, 255) -33.497%,rgba(0, 33, 255, 0) 281.515%,rgb(10, 40, 244) 281.515%,rgb(10, 40, 244) 281.515%);
}

.ton-price {
  color: rgb(255, 255, 255);
  font-family: 'Outfit', sans-serif;
  font-size: 20px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: right;
}

.ton-list {
  height: 25px;
  display: flex;
  align-items: center;
}

.ton-procent {
  color: rgb(109, 188, 53);
  font-family: 'Outfit', sans-serif;
  font-size: 12px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: right;
  width: 22px;
  height: 24px;
  display: flex;
  align-items: center;
}
</style>