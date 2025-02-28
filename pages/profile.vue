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
              <div style="display: flex; justify-content: center;">
                <img style="margin: 20px 0px 0px 0px;" src="public/tonstacker.svg">
              </div>
              <div style="margin: 70px 0px 0px 0px;">
                <div class="profile-info-block">
                  <img class="profile-img" src="public/user.svg" alt="User">
                  <p class="profile-name">{{ userName }}</p>
                  <p class="profile-email">{{ userEmail }}</p>
                  <div class="profil-set">
                    <div style="margin: 14px 0px 0px 16px;">
                      <button style="display: flex;align-items: center;">
                        <img style="height: 24px; width: 24px;" src="public/support.svg">
                        <p class="profile-set-txt">Help & Support</p>
                      </button>
                    </div>
                    <div style="margin: 10px 0px 0px 16px;">
                      <button style="display: flex;align-items: center;">
                        <img style="height: 24px; width: 24px;" src="public/wallet2.svg">
                        <p class="profile-set-txt">How to create a TON wallet</p>
                      </button>
                    </div>
                    <div style="margin: 10px 0px 0px 16px;">
                      <button style="display: flex;align-items: center;">
                        <img style="height: 24px; width: 24px;" src="public/guid.svg">
                        <p class="profile-set-txt">What is TonStaker</p>
                      </button>
                    </div>
                  </div>
                  <div>
                    <p class="promote-txt">Referral promotion</p>
                    <div class="promote-block">
                      <p class="referral-count-profile">Friends invited: {{ referralCount }}</p>
                        <div class="referral-link-container">
                            <input type="text" class="referral-link-profile" :value="referralLink" readonly />
                            <button @click="copyReferralLink" class="copy-btn-profile">Copy</button>
                        </div>
                        <div>
                          <p class="promotion-txt">Terms of the promotion:</p>
                          <p class="promotion-txt-two">1. Invite 5 friends</p>
                          <p class="promotion-txt-two">2. Your 5 friends should have a TON<br>in staking</p>
                        </div>
                        <div>
                          <p class="reward-txt">Reward:</p>
                          <p class="promotion-txt-two">80 <img style="margin: 0px 0px 0px 2px;" src="public/rewardton.svg"></p>
                        </div>
                    </div>
                    <NuxtLink to="/main">
                      <p class="back-main-txt">Back home page</p>
                  </NuxtLink>
                </div>
                </div>
                
              </div>
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
          this.userEmail = null; // Add this line
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
          this.userEmail = user.email; // Add this line to set the userEmail
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
          this.userEmail = null; // Add this line
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
  background-image: url(public/profile.svg);
  width: 100%;
  height: 100vh;
  background-position: center; /* Фиксирует изображение по центру */
  background-size: cover; /* Масштабирует изображение так, чтобы оно покрывало всю область, обрезая лишнее */
  background-repeat: no-repeat; 
  justify-content: flex-start;
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



.profile-info-block {
    width: 100%;
    height: 100vh;
    background: rgb(255, 255, 255);
    display: flex;
    flex-direction: column;
    align-items: center;
}

.profile-img {
    width: 120px;
    height: 120px;
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
    color: #17181C;
    font-family: 'Montserrat', sans-serif;
    font-size: 24px;
    font-weight: 700;
    line-height: 32px;
    letter-spacing: 0%;
    text-align: center;
    margin: 100px 0px 0px 0px;
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
    width: 335px;
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

.referral-count-profile {
  background: linear-gradient(142.59deg, rgb(29, 97, 231),rgb(62, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 24px;
  letter-spacing: -1%;
  text-align: left;
  margin: 19px 0px 0px 0px;
  width: 283px;
}

.referral-link-container {
  display: flex;
  align-items: center;
  margin-bottom: 5px;
  width: 283px;
}

.referral-link-profile {
  flex: 1;
  box-sizing: border-box;
  border: 2px solid rgb(237, 241, 243);
  border-radius: 2px;
  background: rgb(255, 255, 255);
  border-radius: 4px;
  color: black;
  margin: 0px;
  padding: 5px 0px 5px 5px;
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  width: 200px;
  height: 26px;
}

.copy-btn-profile {
  border-radius: 4px;
  border: none;
  color: #FFFFFF;
  background: linear-gradient(132.02deg, rgb(29, 97, 231) 0%,rgb(62, 119, 234) 110.341%);
  font-family: 'Montserrat', sans-serif;
  font-size: 12px;
  cursor: pointer;
  width: 74px;
  height: 26px;
  margin: 0px 0px 0px 9px;
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

.profil-set {
  border-radius: 8px;
  box-shadow: 0px 1px 4px 0px rgba(0, 0, 0, 0.25);
  background: rgb(255, 255, 255);
  width: 327px;
  height: 121px;
  margin: 14px 0px 0px 0px;
}

.profile-set-txt {
  color: #17181C;
  font-family: 'Montserrat', sans-serif;
  font-size: 14px;
  font-weight: 600;
  margin-left: 13px;
  text-align: center;
}

.promote-block {
  background-image: url(public/backton.svg);
  width: 335px;
  height: 225px;
  background-position: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.promotion-txt {
  background: linear-gradient(142.59deg, rgb(29, 97, 231),rgb(62, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 24px;
  letter-spacing: -1%;
  text-align: left;
  width: 283px;
  margin: 20px 0px 0px 0px;
}

.promotion-txt-two {
  color: #17181C;
  font-family: 'Inter';
  font-size: 12px;
  font-weight: 500;
  line-height: 15px;
  letter-spacing: -1%;
  text-align: left;
  width: 283px;
  display: flex;
  transform: translateX(15px);
}

.reward-txt {
  background: linear-gradient(142.59deg, rgb(29, 97, 231),rgb(62, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  line-height: 24px;
  letter-spacing: -1%;
  text-align: left;
  width: 283px;
  margin: 5px 0px 0px 0px;
}

.back-main-txt {
  color: #17181C;
  font-family: 'Montserrat', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0%;
  text-align: center;
  width: 295px;
  height: 17px;
  margin: 4px 0px 0px 0px;
}
</style>