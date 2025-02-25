<template>
    <div>
        <div class="backround">
            <div class="request-form">
                <h3>Отправить запрос</h3>
                <form @submit.prevent="submitRequest">
                    <div>
                        <label for="productName">Название товара:</label>
                        <input type="text" id="productName" v-model="requestForm.productName" required>
                    </div>
                    <button type="submit">Отправить</button>
                </form>
                <p v-if="requestSuccess">Запрос успешно отправлен!</p>
                <p v-if="requestError">{{ requestError }}</p>
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
      amount: 180,
      requestForm: {
        productName: ''
      },
      requestSuccess: false,
      requestError: null
    };
  },
  methods: {
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

<style scoped>
.request-form {
  margin-top: 20px;
}
.request-form h3 {
  margin-bottom: 10px;
}
.request-form label {
  display: block;
  margin-bottom: 5px;
}
.request-form input {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  box-sizing: border-box;
}
.request-form button {
  padding: 8px 16px;
}
.request-form p {
  margin-top: 10px;
}
</style>