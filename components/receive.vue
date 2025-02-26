<template>
    <div>
      <div class="backround-recive">
        <form class="form-settings" id="registration-form" @submit.prevent="sendReceiveData">
          <div class="register-form">
            <p class="register-txt">Send TON</p>
  
            <p class="form-placholder">Email</p>
            <input 
              type="email" 
              id="email" 
              name="email" 
              v-model="email" 
              required
              class="form-block"
            />
  
            <p class="form-placholder">Wallet Address</p>
            <input 
              type="text" 
              id="wallet-address" 
              name="wallet-address" 
              v-model="walletAddress" 
              required
              class="form-block"
            />
          </div>
  
          <button class="register-btn" type="submit">Отправить</button>
        </form>
        <NuxtLink to="/main">
            <p class="auth-txt">Back home page</p>
        </NuxtLink>
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
        email: '',
        walletAddress: ''
      };
    },
    methods: {
      async sendReceiveData() {
        try {
          const { data, error } = await supabase
            .from('receive')
            .insert([
              {
                email: this.email,
                wallet_address: this.walletAddress,
              },
            ]);
  
          if (error) {
            console.error('Ошибка отправки данных:', error);
          } else {
            console.log('Данные отправлены успешно:', data);
          }
        } catch (error) {
          console.error('Общая ошибка при отправке данных:', error);
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
  
  .backround-recive {
    background: rgb(12, 13, 14);
    width: 100%;
    height: 1200px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
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
  
  .html {
    touch-action: manipulation; /* Отключает двойной тап и зум */
  }
</style>