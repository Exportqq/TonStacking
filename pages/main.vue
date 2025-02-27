<template>
  <Head>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, user-scalable=no">
  </Head>

  <div>
    <!-- Показываем спиннер загрузки с анимацией fade -->
    <Transition name="fade">
      <div v-if="isLoading" class="loading-container">
        <div id="app">
          <atom-spinner
            :animation-duration="1200"
            :size="100"
            :color="'#25A3E2'"
          />
        </div>
      </div>
    </Transition>

    <!-- Показываем основной контент с анимацией fade-cont -->
    <Transition name="fade-cont">
      <div v-if="!isLoading">
        <div class="backround">
          <div class="header-navigation">
            <div class="header-block">
              <div style="display: flex; align-items: center; margin-top: 15px;">
                <NuxtLink to="/profile">
                  <div style="float: left;">
                    <img src="public/user.svg" alt="User">
                  </div>
                  <div style="float: left;">
                    <p class="user-name">{{ userName }}</p>
                  </div>
                </NuxtLink>            
                <button @click="logoutUser"><img class="log-out-img" src="public/logout.svg" alt="Logout"></button>
              </div>
            </div>
          </div>

          <div>
            <p class="balance">{{ userBalance }} TON</p>
            <p class="balance-txt-status" v-if="userBalance >= 100 && userBalance <= 249">Staking will end in 31 days</p>
            <p class="balance-txt-status" v-if="userBalance >= 250 && userBalance <= 319">Staking will end in 21 days</p>
            <p class="balance-txt-status" v-if="userBalance >= 320">Staking will end in 14 days</p>
          </div>


        <div class="backroud-all-block">
          <div class="transaction-navigation">
            <NuxtLink to="/sendTON">
              <button class="transaction-btn">
                <img src="public/send.svg" alt="Send">
                <p class="transaction-txt">Send</p>
              </button>
            </NuxtLink>
            <div class="vertical-line"></div>
            <NuxtLink to="/lowTON">
              <button class="transaction-btn">
                <img src="public/wallet.svg" alt="Receive">
                <p class="transaction-txt">Receive</p>
              </button>
            </NuxtLink>
         </div>
          <div class="staking-header">
            <div class="horizontal-line"></div>
            <p class="stake-txt">Staking</p>
            <div class="horizontal-line-two"></div>
          </div>
          <div style="display: flex; flex-direction: column; align-items: center; overflow: scroll;">
            <div class="stake-block">
              <div style="float: left;">
                <p class="stake-block-txt">Staking for 31 days</p>
                <NuxtLink to="/lowTON">
                  <button class="stake-block-btn">Stake</button>
                </NuxtLink>
              </div>
              <div style="float: left;">
                <div style="display: flex; flex-direction: column; align-items: center;">
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">100</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                  <ul style="margin: 3px 24px 0px 0px;">
                    <li>
                      <p class="ton-procent">36%</p>
                    </li>
                    <li>
                      <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg" alt="Swap">
                    </li>
                  </ul>
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">136</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <div class="stake-block">
              <div style="float: left;">
                <p class="stake-block-txt">Staking for 21 days</p>
                <NuxtLink to="/middleTON">
                  <button class="stake-block-btn">Stake</button>
                </NuxtLink>
              </div>
              <div style="float: left;">
                <div style="display: flex; flex-direction: column; align-items: center;">
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">250</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                  <ul style="margin: 3px 24px 0px 0px;">
                    <li>
                      <p class="ton-procent">44%</p>
                    </li>
                    <li>
                      <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg" alt="Swap">
                    </li>
                  </ul>
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">360</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <div class="stake-block">
              <div style="float: left;">
                <p class="stake-block-txt">Staking for 14 days</p>
                <NuxtLink to="/highTON">
                  <button class="stake-block-btn">Stake</button>
                </NuxtLink>
              </div>
              <div style="float: left;">
                <div style="display: flex; flex-direction: column; align-items: center;">
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">320</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                  <ul style="margin: 3px 24px 0px 0px;">
                    <li>
                      <p class="ton-procent">51%</p>
                    </li>
                    <li>
                      <img style="margin: 0px 0px 0px 2px;" src="public/swap.svg" alt="Swap">
                    </li>
                  </ul>
                  <ul class="ton-list">
                    <li>
                      <p class="ton-price">485</p>
                    </li>
                    <li>
                      <img style="margin: 2px 0px 0px 4px;" src="public/path.svg" alt="Path">
                    </li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </div>
        </div>
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
      isLoading: true, // Добавляем переменную для состояния загрузки
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
      referrerCode: ''
    };
  },
  name: 'register',
  async asyncData({ query }) {
    const ref = query.ref
    // Обработка параметра ref
  },
  mounted() {
    this.checkSession();
    const urlParams = new URLSearchParams(window.location.search);
    this.referrerCode = urlParams.get('ref') || '';
  },
  components: {
    AtomSpinner
  },
  methods: {
    // Вспомогательная функция для минимальной задержки
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },

    async checkSession() {
      try {
        this.isLoading = true;
        
        const sessionPromise = supabase.auth.getSession();
        const delayPromise = this.delay(2000);

        const [{ data: { session }, error: sessionError }] = await Promise.all([
          sessionPromise,
          delayPromise
        ]);

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

    async copyReferralLink() {
      try {
        await navigator.clipboard.writeText(this.referralLink);
        alert('Реферальная ссылка скопирована!');
      } catch (err) {
        console.error('Не удалось скопировать ссылку: ', err);
        alert('Не удалось скопировать ссылку. Попробуйте скопировать вручную.');
      }
    },

    generateReferralCode() {
      return Math.random().toString(36).substring(2, 10);
    },

    async registerUser() {
      this.isLoading = true;
      this.registrationError = null;
      this.registrationSuccess = false;

      try {
        const registerPromise = supabase.auth.signUp({
          email: this.registrationForm.email,
          password: this.registrationForm.password,
          options: {
            data: {
              full_name: this.registrationForm.name,
            }
          }
        });
        const delayPromise = this.delay(2000);

        const [{ error, data }] = await Promise.all([
          registerPromise,
          delayPromise
        ]);

        if (error) {
          console.error('Ошибка регистрации:', error);
          this.registrationError = error.message;
        } else {
          console.log('Регистрация успешна:', data);
          this.registrationSuccess = true;
          
          await this.createUserProfile(data.user.id, this.registrationForm.name);
          
          if (this.referrerCode) {
            await this.incrementReferralCount(this.referrerCode);
          }
          
          this.registrationForm = { name: '', email: '', password: '' };
        }
      } catch (error) {
        console.error('Общая ошибка при регистрации:', error);
        this.registrationError = 'Произошла ошибка при регистрации. Пожалуйста, попробуйте позже.';
      } finally {
        this.isLoading = false;
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

    async incrementReferralCount(referralCode) {
      try {
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
      this.isLoading = true;
      this.loginError = null;
      try {
        const loginPromise = supabase.auth.signInWithPassword({
          email: this.loginForm.email,
          password: this.loginForm.password,
        });
        const delayPromise = this.delay(2000);

        const [{ error, data }] = await Promise.all([
          loginPromise,
          delayPromise
        ]);

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
      } finally {
        this.isLoading = false;
      }
    },

    async logoutUser() {
      this.isLoading = true;
      const router = useRouter();
      try {
        const logoutPromise = supabase.auth.signOut();
        const delayPromise = this.delay(2000);

        const [{ error }] = await Promise.all([
          logoutPromise,
          delayPromise
        ]);

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
          router.push({ path: "/" });
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
  background: rgb(29, 97, 231);
}

ul li {
  list-style: none;
  float: left;
}

button {
  background: none;
  border: none;
}

.backroud-all-block {
  border-radius: 20px 20px 0px 0px;
  background: rgb(255, 255, 255);
  width: 100%;
  height: 100%;
  margin: 20px 0px 0px 0px ;
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
  display: flex;
  align-items: center;
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
  height: 50px;
  margin: 59px 0px 0px 0px;
}

.transaction-txt {
  width: 100%;
  height: 20px;
  color: rgb(23, 24, 28);
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
  box-sizing: border-box;
  border: 1px solid rgb(23, 24, 28);
  border-radius: 24px;
  box-shadow: -4px -3px 0px 0px rgb(29, 97, 231),0px 0px 18px 0px rgba(0, 0, 0, 0.25);
  background: rgb(255, 255, 255);
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
  border-radius: 4px;
  background: linear-gradient(120.09deg, rgb(29, 97, 231) -33.497%,rgba(0, 33, 255, 0) 281.515%,rgb(10, 40, 244) 281.515%,rgb(29, 97, 231) 281.515%);
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

a {
  text-decoration: none;
}

.horizontal-line {
  flex-grow: 1; /* Линии занимают все доступное пространство */
  height: 1px;
  background: linear-gradient(90.00deg, rgb(29, 97, 231),rgba(14, 21, 74, 0) 100%);
}

.horizontal-line-two {
  transform: rotate(180deg);
  flex-grow: 1; /* Линии занимают все доступное пространство */
  height: 1px;
  background: linear-gradient(90.00deg, rgb(29, 97, 231),rgba(14, 21, 74, 0) 100%);
}

.horizontal-line:last-child {
  /* Разворачиваем градиент для правой линии */
  background: linear-gradient(270deg, rgb(21, 53, 244), rgba(14, 21, 74, 0) 100%);
}

.stake-txt {
  color: rgb(23, 24, 28);
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
  box-sizing: border-box;
  border: 1px solid rgb(23, 24, 28);
  border-radius: 16px;
  background: rgb(255, 255, 255);
  display: flex;
  align-items: center;
  margin: 15px 0px 0px 0px;
}

.stake-block-txt {
  color: #17181C;
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
  border-radius: 4px;
  background: linear-gradient(120.09deg, rgb(29, 97, 231) -33.497%,rgba(0, 33, 255, 0) 281.515%,rgb(10, 40, 244) 281.515%,rgb(29, 97, 231) 281.515%);
}

.ton-price {
  color: #17181C;
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

.balance-txt-status {
  color: rgb(255, 255, 255);
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 600;
  line-height: 140%;
  letter-spacing: -1%;
  text-align: center;
}

/* Добавляем стили для центрирования спиннера */
.loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    width: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background-color: #FFFFFF; /* или любой другой цвет фона */
    z-index: 9999;
}
</style>