<template>
    <div class="All_register">
        <div class="login-container">
            <h1>Login</h1>
            <form @submit.prevent="handleLogin" class="login-form">
                <div class="form-group">
                    <label for="username">Username:</label>
                    <input v-model="username" id="username" type="text" autocomplete="username" />
                </div>
                <div class="form-group">
                    <label for="password">Password:</label>
                    <input v-model="password" id="password" type="password" autocomplete="current-password" />
                </div>
                <button type="submit" class="login-button">Login</button>
                <p v-if="error" class="error-message">{{ error }}</p>
                <NuxtLink to="/register">
                    <button type="button" class="register-button">Register</button>
                </NuxtLink>
            </form>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const username = ref('');
const password = ref('');
const error = ref('');

const handleLogin = async () => {
    try {
        const response = await fetch('http://localhost:3000/users');
        const users = await response.json();
        const user = users.find(
            (u) => u.username === username.value && u.password === password.value
        );
        if (user) {
            localStorage.setItem('user', JSON.stringify(user));
            router.push('/main');
        } else {
            error.value = 'Invalid username or password';
        }
    } catch (err) {
        error.value = 'An error occurred. Please try again.';
    }
};
</script>

<style lang="css" scoped>
.All_register {
    max-width: 1080px;
    width: 100%;
    margin: 0 auto;
}

.login-container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
    /* белый фон */
}

h1 {
    text-align: center;
    color: #4caf50;
    /* зеленый цвет заголовка */
}

.login-form {
    display: flex;
    flex-direction: column;
}

.form-group {
    margin-bottom: 15px;
}

label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    color: #388e3c;
    /* темно-зеленый цвет текста */
}

input {
    width: 100%;
    padding: 8px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.login-button,
.register-button {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    background-color: #4caf50;
    /* зеленый цвет кнопки входа */
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.register-button {
    background-color: #81c784;
    /* светло-зеленый цвет кнопки регистрации */
}

.error-message {
    color: red;
    text-align: center;
    margin-top: 10px;
}
</style>
