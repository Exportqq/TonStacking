<template>
<div>
    <div class="backround">
        <form class="form-settings" id="registration-form" @submit.prevent="submitRequest">
            <div class="register-form">
                <p class="register-txt">Transaction</p>

                <p class="form-placholderr">Transfer address</p>
                <div class="form-block">
                    <span class="shortened-address">{{ shortenedAddress }}</span>
                    <p class="copy-text" @click="copyAddress">Скопировать адрес</p>
                    <span ref="fullAddress" style="display: none;">UQCVsvwpa2QptQVsA4thnmxjuxitBDy-tz8oaQm-sm15ba7S</span>
                </div>

                <p class="form-placholders">Sender's address</p>
                <input 
                    type="text" 
                    id="productName" 
                    name="login-password" 
                    v-model="requestForm.productName" 
                    required
                    class="form-block"
                />


                <div style="width: 276px;">
                    <p class="count-txt">Quantity:</p>
                    <p class="txt-variable">{{ amount }}<img src="public/path.svg"></p>
                </div>

            </div>


            <button class="register-btn" type="submit">Done</button>
            <NuxtLink to="/main">
                <p class="auth-txt">Back home page</p>
            </NuxtLink>
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
      fullAddressText: 'UQCVsvwpa2QptQVsA4thnmxjuxitBDy-tz8oaQm-sm15ba7S',
      shortenedAddress: '',
      amount: 180,
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
          alert('Адрес скопирован!'); // Можно заменить на более красивое уведомление
        })
        .catch(err => {
          console.error('Не удалось скопировать адрес:', err);
        });
    },
    async submitRequest() {
      this.requestError = null;
      this.requestSuccess = false;

      try {
        const user = await supabase.auth.getUser();
        if (!user.data.user) {
          throw new Error("Пользователь не авторизован");
        }

        const { error } = await supabase
          .from('user_requests')
          .insert([
            {
              user_email: user.data.user.email,
              product_name: this.requestForm.productName,
              amount: this.amount // Используем фиксированное значение amount
            }
          ]);

        if (error) {
          console.error('Ошибка при отправке запроса:', error);
          this.requestError = error.message;
        } else {
          console.log('Запрос успешно отправлен');
          this.requestSuccess = true;
          this.requestForm.productName = ''; // Очищаем поле формы
        }
      } catch (error) {
        console.error('Общая ошибка при отправке запроса:', error);
        this.requestError = 'Произошла ошибка при отправке запроса. Пожалуйста, попробуйте позже.';
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

.backround {
    background: rgb(12, 13, 14);
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

.form-placholders {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: left;
  height: 21px;
  width: 284px;
  margin: 45px 0px 8px 0px;
}

.form-placholderr {
  color: rgb(255, 255, 255);
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
  font-weight: 500;
  line-height: 32px;
  letter-spacing: 0%;
  text-align: left;
  height: 21px;
  width: 284px;
  margin: 0px 0px 8px 0px;
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

.html {
  touch-action: manipulation; /* Отключает двойной тап и зум */
} 


.form-block {
  display: flex;
  flex-direction: column; /* Размещаем элементы вертикально */
  align-items: flex-start; /* Выравниваем по левому краю */
  border: 1px solid #ccc;
  padding: 8px;
  border-radius: 4px;
}

.shortened-address {
  font-size: 1.1em;
  margin-bottom: 5px;
  word-break: break-all;
}

.copy-text {
  font-size: 12px;
  color: #007bff;
  cursor: pointer;
  text-decoration: underline;
}

.copy-text:hover {
  color: #0056b3;
}

.count-txt {
    color: rgb(255, 255, 255);
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

.txt-variable {
    color: rgb(255, 255, 255);
    font-family: 'Outfit', sans-serif;
    font-size: 20px;
    font-weight: 500;
    line-height: 32px;
    letter-spacing: 0%;
    width: 100%;
    height: 16px;
}
</style>