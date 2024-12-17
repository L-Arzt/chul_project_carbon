<template>
    <div class="All_register">
        <div class="register-container">
            <h1>Register</h1>
            <form @submit.prevent="handleRegister" class="register-form">
                <div class="form-group">
                    <label for="username">Username:</label>
                    <input v-model="username" id="username" type="text" />
                </div>
                <div class="form-group">
                    <label for="password">Password:</label>
                    <input v-model="password" id="password" type="password" />
                </div>
                <button type="submit" class="register-button">Register</button>
                <p v-if="error" class="error-message">{{ error }}</p>
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

const handleRegister = async () => {
    if (!username.value || !password.value) {
        error.value = 'Username and password are required';
        return;
    }

    try {
        const response = await fetch('http://localhost:3000/users');
        const users = await response.json();
        const userExists = users.some((u) => u.username === username.value);

        if (userExists) {
            error.value = 'Username already exists';
        } else {
            const newUser = {
                id: users.length ? users[users.length - 1].id + 1 : 1,
                username: username.value,
                password: password.value,
                results: [], // Инициализация массива результатов
            };

            await fetch('http://localhost:3000/users', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(newUser),
            });

            localStorage.setItem('user', JSON.stringify(newUser));
            router.push('/');
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

.register-container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
    /* Белый фон */
}

h1 {
    text-align: center;
    color: #4caf50;
    /* Зеленый цвет заголовка */
}

.register-form {
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
    /* Темно-зеленый цвет текста */
}

input {
    width: 100%;
    padding: 8px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.register-button {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    background-color: #4caf50;
    /* Зеленый цвет кнопки регистрации */
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.error-message {
    color: red;
    text-align: center;
    margin-top: 10px;
}
</style>
