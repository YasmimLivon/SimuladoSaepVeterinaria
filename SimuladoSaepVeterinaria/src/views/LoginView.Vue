<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

const credenciais = reactive({
  username: '',
  password: ''
});

const erroLogin = ref('');
const token = ref('');
const carregando = ref(false);

const realizarLogin = async () => {
  carregando.value = true;
  erroLogin.value = '';
  token.value = '';

  try {
    const resposta = await fetch('http://localhost:5119/api/Auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        username: credenciais.username,
        password: credenciais.password
      })
    });

    if (!resposta.ok) {
      throw new Error('Falha na autenticação. Verifique suas credenciais.');
    }

    const dados = await resposta.json();
    token.value = dados.token;
    localStorage.setItem('token', dados.token);

    router.push('/produtos');

  } catch (erro) {
    if (erro instanceof Error) {
      erroLogin.value = erro.message;
    } else {
      erroLogin.value = 'Ocorreu um erro desconhecido.';
    }
  } finally {
    carregando.value = false;
  }
}
</script>

<template>
  <div class="login-container">
    <h2>Acesso ao Sistema</h2>

    <form @submit.prevent="realizarLogin">
      <div class="campo">
        <label for="username">Usuário:</label>
        <input 
          type="text" 
          id="username" 
          v-model="credenciais.username" 
          required
        >
      </div>

      <div class="campo">
        <label for="password">Senha:</label>
        <input 
          type="password" 
          id="password" 
          v-model="credenciais.password" 
          required
        >
      </div>

      <button type="submit" :disabled="carregando">
        {{ carregando ? 'Entrando...' : 'Entrar' }}
      </button>

      <p v-if="erroLogin" class="mensagem-erro">{{ erroLogin }}</p>
      <p v-if="token" class="mensagem-sucesso">Token recebido com sucesso!</p>
    </form>
  </div>
</template>

<style scoped>
.login-container {
  max-width: 350px;
  margin: 80px auto;
  padding: 24px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.campo {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
}

.campo label {
  margin-bottom: 5px;
  font-weight: bold;
}

.campo input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #42b883;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #a0d6bc;
  cursor: not-allowed;
}

.mensagem-erro {
  color: red;
  font-size: 14px;
  margin-top: 10px;
}

.mensagem-sucesso {
  color: green;
  font-size: 14px;
  margin-top: 10px;
}
</style>