<template>
    <div class="container" v-if="loggedInUser">
        <h1>Калькулятор углеродного следа</h1>
        <div class="results">
            <h2>Ваши результаты:</h2>
            <ul class="results-list">
                <li v-for="(result, index) in loggedInUser.results" :key="index">
                    Итог: {{ result }} кг/м³
                </li>
            </ul>
        </div>
        <form @submit.prevent="calculateCarbonFootprint">
            <div class="question" v-if="currentQuestion < questions.length && questions.length > 0">
                <h3>{{ questions[currentQuestion].text }}</h3>
                <div class="options">
                    <label v-for="(option, idx) in questions[currentQuestion].options" :key="idx" class="option-label">
                        <input type="radio" :name="'question' + currentQuestion" :value="option.value"
                            v-model="answers[currentQuestion]" required>
                        {{ option.text }} ({{ option.co2 }} кг/м³)
                    </label>
                </div>
                <button type="button" @click="nextQuestion" :disabled="!answers[currentQuestion]">Далее</button>
            </div>
            <div v-if="carbonFootprint !== null" class="result">
                <h2>Итог: {{ carbonFootprint }} кг/м³</h2>
                <button type="button" @click="saveResult">Сохранить результат</button>
            </div>
        </form>
    </div>
    <div v-else>
        <h2>Пожалуйста, войдите в систему, чтобы использовать калькулятор</h2>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

// Статическое определение вопросов
const questions = ref([
    {
        "text": "Какой вид топлива вы используете для отопления вашего дома?",
        "options": [
            { "text": "Газ", "value": 0.2, "co2": "0,2" },
            { "text": "Электричество", "value": 0.5, "co2": "0,5" },
            { "text": "Дрова", "value": 0.4, "co2": "0,4" }
        ]
    },
    {
        "text": "Какова ваша средняя дистанция поездок на автомобиле в неделю?",
        "options": [
            { "text": "Менее 50 км", "value": 0.1, "co2": "0,1" },
            { "text": "От 50 до 200 км", "value": 0.3, "co2": "0,3" },
            { "text": "Более 200 км", "value": 0.5, "co2": "0,5" }
        ]
    },
    {
        "text": "Как часто вы пользуетесь общественным транспортом?",
        "options": [
            { "text": "Каждый день", "value": 0.1, "co2": "0,1" },
            { "text": "Несколько раз в неделю", "value": 0.3, "co2": "0,3" },
            { "text": "Редко или никогда", "value": 0.5, "co2": "0,5" }
        ]
    },
    {
        "text": "Какое количество пищи вы покупаете в неделю?",
        "options": [
            { "text": "Менее 10 кг", "value": 0.2, "co2": "0,2" },
            { "text": "От 10 до 30 кг", "value": 0.4, "co2": "0,4" },
            { "text": "Более 30 кг", "value": 0.6, "co2": "0,6" }
        ]
    },
    {
        "text": "Как часто вы используете пластиковые упаковки?",
        "options": [
            { "text": "Редко", "value": 0.1, "co2": "0,1" },
            { "text": "Иногда", "value": 0.3, "co2": "0,3" },
            { "text": "Часто", "value": 0.5, "co2": "0,5" }
        ]
    },
    {
        "text": "Какова ваша привычка по утилизации отходов?",
        "options": [
            { "text": "Сортирую и перерабатываю", "value": 0.1, "co2": "0,1" },
            { "text": "Утилизирую, но не сортирую", "value": 0.3, "co2": "0,3" },
            { "text": "Не обращаю на это внимания", "value": 0.5, "co2": "0,5" }
        ]
    },
    {
        "text": "Как часто вы покупаете новую одежду?",
        "options": [
            { "text": "Реже одного раза в месяц", "value": 0.1, "co2": "0,1" },
            { "text": "Один раз в месяц", "value": 0.3, "co2": "0,3" },
            { "text": "Чаще одного раза в месяц", "value": 0.5, "co2": "0,5" }
        ]
    }
]);

const answers = ref(new Array(7).fill(null));
const currentQuestion = ref(0);
const carbonFootprint = ref(null);
const loggedInUser = ref(null);
const router = useRouter();

const loadUserData = () => {
    const user = JSON.parse(localStorage.getItem('user'));
    if (user) {
        loggedInUser.value = user;
    }
};

const nextQuestion = () => {
    if (currentQuestion.value < questions.value.length - 1) {
        currentQuestion.value++;
    } else {
        calculateCarbonFootprint();
    }
};

const calculateCarbonFootprint = () => {
    carbonFootprint.value = answers.value.reduce((total, answer) => total + (answer || 0), 0).toFixed(2);
};

const saveResult = async () => {
    if (loggedInUser.value && carbonFootprint.value) {
        try {
            loggedInUser.value.results.push(carbonFootprint.value);
            await fetch(`http://localhost:3000/users/${loggedInUser.value.id}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(loggedInUser.value),
            });
        } catch (error) {
            console.error('Ошибка при сохранении результата:', error);
        }
    }
};

onMounted(loadUserData);
</script>

<style scoped>
.container {
    max-width: 600px;
    margin: auto;
    padding: 20px;
    font-family: Arial, sans-serif;
}

.results-list {
    list-style-type: none;
    /* Убираем маркеры списка */
    padding: 0;
    /* Убираем отступы */
}

.results-list li {
    margin-bottom: 10px;
    /* Отступ между элементами списка */
    font-weight: bold;
    /* Делаем текст жирным */
}

.question {
    border: 2px solid #4CAF50;
    border-radius: 8px;
    padding: 20px;
    margin-bottom: 20px;
    background-color: #f9f9f9;
}

.options {
    margin-top: 10px;
}

.option-label {
    display: block;
    /* Отображаем каждый вариант ответа в отдельной строке */
    margin-bottom: 10px;
    /* Отступ между вариантами ответа */
}

button {
    background-color: #4CAF50;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 5px;
    cursor: pointer;
}

button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

.result {
    margin-top: 20px;
    font-weight: bold;
}
</style>
