<template>
    <div>
        <!-- Показываем только спиннер, если идет загрузка -->
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
            <div v-if="!isLoading" class="profile-backround">
                <div class="profile-block">
                    <img class="profile-img" src="public/user.svg" alt="User">
                    <p class="profile-name">{{ userName }}</p>
                    <p class="profile-email">{{ userEmail }}</p>
                    <div class="support">
                        <a href="/">
                            <div style="display: flex; align-items: center;">
                                <img style="margin-right: 8px;" src="public/telegram.svg" alt="Telegram">
                                <p class="support-txt">Support</p>
                                <img style="margin-left: 8px;" src="public/arrow.svg" alt="Arrow">
                            </div>
                        </a>
                    </div>
                    <p class="promote-txt">Referral promotion</p>
                    <div class="promote-block">
                        <div style="height: 20px; display: flex; width: 120px; align-items: center; margin: 0 0 0 43px;">
                            <div style="margin: 2px 2px 0px 0px;">
                                <img src="public/ton.svg" alt="Path">
                            </div>
                            <div>
                                <p class="promote-name">Promotion</p>
                            </div>
                        </div>
                        <div>
                            <p class="promote-comment">If you invite 5 friends who stake their assets, you will receive 150 TON</p>
                        </div>
                    </div>

                    <div class="referral-block">
                        <p class="referral-title">Invite friends</p>
                        <p class="referral-count">Friends invited: {{ referralCount }}</p>
                        <div class="referral-link-container">
                            <input type="text" class="referral-link" :value="referralLink" readonly />
                            <button @click="copyReferralLink" class="copy-btn">Copy</button>
                        </div>
                    </div>
                </div>  
                <NuxtLink to="/main">
                    <p class="auth-txt">Back home page</p>
                </NuxtLink>
            </div>
        </Transition>
    </div>
</template>

<script>
import {AtomSpinner} from 'epic-spinners'
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = "https://dvdpezcwkklhlxafpyfl.supabase.co";
const supabaseKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR2ZHBlemN3a2tsaGx4YWZweWZsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAxNjE0MDcsImV4cCI6MjA1NTczNzQwN30.lhw8XGLjw2GBhSLwuUaMGlz67vt9k1MztLUnRE7qBGM";
const supabase = createClient(supabaseUrl, supabaseKey);

export default {
  data() {
    return {
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
      const router = useRouter()
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
          router.push({ path: "/login" })
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
    margin: 0px;
}

.profile-backround {
    background: #1D61E7;
    width: 100%;
    height: 1200px;
    display: flex;
    justify-content: center;
    display: flex;
    justify-content: flex-start;
    flex-direction: column;
    align-items: center;
}

.profile-block {
    width: 335px;
    height: 560px;
    border-radius: 32px;
    box-sizing: border-box;
    border: 2px solid rgb(237, 241, 243);
    border-radius: 32px;
    background: rgb(255, 255, 255);
    margin: 120px 0px 0px 0px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.profile-img {
    width: 68px;
    height: 68px;
    transform: translateY(-34px);
    margin: 0px 0px 0px 0px;
    position: absolute;
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


.profile-name {
    color: rgb(23, 24, 28);
    font-family: 'Montserrat', sans-serif;
    font-size: 24px;
    font-weight: 700;
    line-height: 32px;
    letter-spacing: 0%;
    text-align: center;
    margin: 48px 0px 0px 0px;
}

.profile-email {
    color: rgb(106, 111, 135);
    font-family: 'Nunito Sans', sans-serif;
    font-size: 14px;
    font-weight: 400;
    line-height: 20px;
    letter-spacing: -1%;
    text-align: center;
}

.support {
    border-radius: 20px;
    width: 295px;
    height: 68px;
    box-sizing: border-box;
    border: 1px solid rgb(237, 241, 243);
    border-radius: 20px;
    box-shadow: -4px -3px 0px 0px rgb(37, 163, 226);
    background: rgb(255, 255, 255);
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 32px 0px 0px 0px;
}

.support-txt {
    /* Body/16/Semibold */
    color: rgb(23, 24, 28);
    font-family: 'Nunito Sans', sans-serif;
    font-size: 16px;
    font-weight: 600;
    line-height: 24px;
    letter-spacing: -1%;
    text-align: left;
    width: 59px;
    height: 44px;
    margin: 0px 110px 0px 12px;
}

.promote-txt {
    color: rgb(23, 24, 28);
    font-family: 'Montserrat', sans-serif;
    font-size: 16px;
    font-weight: 600;
    line-height: 20px;
    letter-spacing: -1%;
    text-align: center;
    width: 295px;
    height: 26px;
    margin: 36px 0px 0px 0px;
}

.promote-name {
    color: rgb(255, 255, 255);
    font-family: 'Montserrat', sans-serif;
    font-size: 16px;
    font-weight: 600;
    line-height: 20px;
    letter-spacing: -1%;
    text-align: center;
    width: 87px;
    height: 20px;
}

.promote-block {
    background-image: url(public/tons.png);
    width: 295px;
    height: 114px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    border-radius: 6px;
}

.promote-comment {
    color: rgb(255, 255, 255);
    font-family: 'Nunito Sans', sans-serif;
    font-size: 12px;
    font-weight: 600;
    line-height: 14px;
    letter-spacing: 0%;
    text-align: center;
    width: 161px;
    height: 42px;
    margin: 2px 0px 0px 17px;
}

/* Стили для реферальной системы */
.referral-block {
  width: 265px;
  box-sizing: border-box;
  border: 1px solid rgb(237, 241, 243);
  box-shadow: -4px -3px 0px 0px rgb(37, 163, 226);
  background: rgb(255, 255, 255);
  border-radius: 16px;
  padding: 15px;
  margin-top: 20px;
}

.referral-title {
  color: #17181C;
  font-family: 'Montserrat', sans-serif;
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 10px;
  text-align: center;
}

.referral-count {
  color: #17181C;
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
  background: #1D61E7;
  border: none;
  border-radius: 4px;
  padding: 8px;
  color: white;
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  margin-right: 5px;
}

.copy-btn {
  background: #1D61E7;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  cursor: pointer;
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

.loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    width: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background-color: #FFFFFF;
    z-index: 9999;
}

/* Стили для анимации затухания */
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

</style>