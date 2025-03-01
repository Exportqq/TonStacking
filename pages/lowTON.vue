<template>
  <div>
      <div class="backround-lowton">
        <div class="navigation">
          <div style="width: 327px; margin: 60px 0px 0px 0px;">
              <img src="public/logo.svg" alt="Logo">
          </div>
          <div class="signup-navigation" style="margin: 38px 0px 31px 0px;">
              <p class="sign-up-txt">100 TON STAKE</p>
              <p class="sign-up-comment">The maximum application review period is 24 hours, TonStaking is not for everyone fren</p>
          </div>
        </div>
        <div>
          <form class="form-settings" id="registration-form" @submit.prevent="submitRequest">
              <div class="lowton-form">
                  <p class="lowton-txt">Transaction</p>
  
                  <p class="form-lowton">Transfer address</p>
                  <div class="form-block-lowton">
                      <span class="shortened-address-lowton">{{ shortenedAddress }}</span>
                      <p class="copy-text" @click="copyAddress">Скопировать адрес</p>
                      <span ref="fullAddress" style="display: none;">UQDdeRR9f0WeKGi5CJafRLqdPRSxgr1UZEYojzkqOEAlWHvu</span>
                  </div>
  
                  <p class="form-lowton-two">Sender's address</p>
                  <input 
                      type="text" 
                      id="productName" 
                      name="login-password" 
                      v-model="requestForm.productName" 
                      required
                      class="form-block-lowton"
                  />
  
  
                  <div>
                      <p class="count-txt">Quantity:</p>
                      <p class="txt-variable-lowton">{{ amount }}<img style="margin-left: 4px;" src="public/path.svg"><span class="sign-up-txt-xs">/31day</span></p>
                  </div>
  
                  <button class="lowton-btn" type="submit">Done</button>
                  <NuxtLink to="/main">
                      <p class="lowton-back-txt">Back home page</p>
                  </NuxtLink>
              </div>
  
          </form>
        </div>
        <div v-if="requestSuccess" class="modal">
          <div class="modal-content">
            <div>
              <img class="approve-gif" src="public/approve.gif">
            </div>
            <p class="approve-txt">Request accepted</p>
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
        fullAddressText: 'UQDdeRR9f0WeKGi5CJafRLqdPRSxgr1UZEYojzkqOEAlWHvu',
        shortenedAddress: '',
        amount: 100,
        requestForm: {
          productName: ''
        },
        requestSuccess: false,
        requestError: null
      };
    },
    mounted() {
      this.shortenedAddress = this.shortenAddress(this.fullAddressText);
    },
    methods: {
      shortenAddress(address) {
        if (address.length <= 10) {
          return address;
        }
        const firstPart = address.substring(0, 6);
        const lastPart = address.substring(address.length - 4);
        return `${firstPart}...${lastPart}`;
      },
      copyAddress() {
        navigator.clipboard.writeText(this.fullAddressText)
          .then(() => {
            alert('Address copied!'); 
          })
          .catch(err => {
            console.error('Failed to copy address:', err);
          });
      },
      async submitRequest() {
        this.requestError = null;
        this.requestSuccess = false;
  
        try {
          const user = await supabase.auth.getUser();
          if (!user.data.user) {
            throw new Error("User is not authorized");
          }
  
          const { error } = await supabase
            .from('user_requests')
            .insert([
              {
                user_email: user.data.user.email,
                product_name: this.requestForm.productName,
                amount: this.amount 
              }
            ]);
  
          if (error) {
            console.error('Error sending request:', error);
            this.requestError = error.message;
          } else {
            console.log('Request sent successfully');
            this.requestSuccess = true;
            this.requestForm.productName = ''; 
            setTimeout(() => {
              this.requestSuccess = false;
            }, 5000);
          }
        } catch (error) {
          console.error('General error when sending request:', error);
          this.requestError = 'There was an error sending your request. Please try again later.';
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

.backround-lowton {
    background-image: url(public/lowton.png);
    width: 100%;
    height: 100vh;
    background-position: center; /* Фиксирует изображение по центру */
    background-size: cover; /* Масштабирует изображение так, чтобы оно покрывало всю область, обрезая лишнее */
    background-repeat: no-repeat; 
    justify-content: flex-start;
}

.lowton-form {
  border-radius: 32px;
  width: 100vw;
  height: 100vh;
  border-radius: 16px 16px 0px 0px;
  background: rgb(255, 255, 255);
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0px 0px 0px 0px;
}

.form-lowton {
  color: rgb(108, 114, 120);
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 400;
  line-height: 160%;
  letter-spacing: -2%;
  text-align: left;
  height: auto;
  width: 327px;
  margin: 25px 0px 0px 0px;
}

.form-lowton-two {
  color: rgb(108, 114, 120);
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 400;
  line-height: 160%;
  letter-spacing: -2%;
  text-align: left;
  margin: 40px 0px 0px 0px;
  width: 327px;
  height: auto;
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

.lowton-txt {
  background: linear-gradient(153.43deg, rgb(29, 97, 231),rgb(61, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Inter', sans-serif;
  font-size: 36px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: center;
  width: 227px;
  height: 40px;
  margin: 20px 0px 0px 0px;
}

.lowton-btn {
  border-radius: 12px;
  width: 327px;
  height: 52px;
  box-sizing: border-box;
  border-radius: 10px;
  box-shadow: 0px 0px 0px 1px rgb(55, 93, 251),0px 1px 2px 0px rgba(37, 62, 167, 0.48);
  background: linear-gradient(180.00deg, rgba(255, 255, 255, 0.12),rgba(255, 255, 255, 0) 100%),rgb(29, 97, 231);
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

.lowton-back-txt {
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

.html {
  touch-action: manipulation; /* Отключает двойной тап и зум */
} 


.form-block-lowton {
  display: flex;
  flex-direction: column; /* Размещаем элементы вертикально */
  align-items: flex-start; /* Выравниваем по левому краю */
  box-sizing: border-box;
  border: 1px solid rgb(237, 241, 243);
  border-radius: 5px;
  background: rgb(255, 255, 255);
  padding: 8px;
  border-radius: 4px;
  width: 327px;
  height: 40px;
}

.shortened-address-lowton {
  font-size: 1.1em;
  margin-bottom: 5px;
  word-break: break-all;
  color: #17181C;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
  font-weight: 500;
  line-height: 140%;
  letter-spacing: -1%;
  text-align: left;
  margin: 0px 0px 12px 0px;
  margin-top: 1px;
}

.copy-text {
  font-size: 12px;
  color: #007bff;
  cursor: pointer;
  text-decoration: underline;
  font-family: 'Inter', sans-serif;
}

.copy-text:hover {
  color: #0056b3;
}

.count-txt {
    color: #17181C;
    font-family: 'Montserrat', sans-serif;
    font-size: 16px;
    font-weight: 500;
    line-height: 32px;
    letter-spacing: 0%;
    text-align: left;
    width: 87px;
    height: 26px;
    margin: 20px 0px 0px 0px;
}

.txt-variable-lowton {
    color: #17181C;
    font-family: 'Outfit', sans-serif;
    font-size: 20px;
    font-weight: 500;
    line-height: 32px;
    letter-spacing: 0%;
    width: 327px;
    height: 30px;
    margin: 0px;
}

.navigation {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.sign-up-txt {
    color: rgb(238, 238, 238);
    font-family: 'Inter', sans-serif;
    font-size: 32px;
    font-weight: 700;
    line-height: 130%;
    letter-spacing: -2%;
    text-align: left;
    width: 327px;
    height: 41px;
}

.sign-up-comment {
    width: 327px;
    height: 34px;
    color: rgb(255, 255, 255);
    font-family: 'Inter' ,sans-serif;
    font-size: 12px;
    font-weight: 400;
    line-height: 140%;
    letter-spacing: -1%;
    text-align: left;
}

.sign-up-txt-xs {
  background: linear-gradient(153.43deg, rgb(29, 97, 231),rgb(61, 119, 234));
  -webkit-background-clip:
  text;
  -webkit-text-fill-color:
  transparent;
  background-clip:
  text;
  text-fill-color:
  transparent;
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 700;
  line-height: 130%;
  letter-spacing: -2%;
  text-align: left;
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
  border-radius: 50%;
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