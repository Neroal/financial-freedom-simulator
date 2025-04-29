<script setup lang="ts">
import { ref, watchEffect, onMounted, reactive, nextTick } from 'vue';

const isDark = ref(true);

const placeholders = {
    currentAge: 30,
    startingInvestment: 1000000,
    monthlyInvestment: 1000,
    annualLumpSumInvestment: 100000,
    dividendYield: 8,
    assetGrowthRate: 2,
    inflationRate: 2,
    monthlyTargetExpense: 50000
};

const formData = reactive({
    currentAge: null,
    startingInvestment: null,
    monthlyInvestment: null,
    annualLumpSumInvestment: null,
    dividendYield: null,
    assetGrowthRate: null,
    inflationRate: null,
    monthlyTargetExpense: null
});

const resultRows = ref<
    {
        age: number;
        startingInvestment: number;
        monthlyInvestment: number;
        endOfYearAmount: number;
        dividendIncome: number;
        annualTargetExpense: number;
    }[]
>([]);

const showError = ref(false);

function showErrorPopup(message: string) {
    errorMessage.value = message;
    showError.value = true;
    setTimeout(() => {
        showError.value = false;
    }, 2000);
}

const financialFreedomAge = ref<number | null>(null);
const errorMessage = ref('');

function toggleTheme() {
    document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light');
    localStorage.setItem('theme', isDark.value ? 'dark' : 'light');
}

onMounted(() => {
    const saved = localStorage.getItem('theme');
    isDark.value = saved === 'dark';
    toggleTheme();
});

watchEffect(toggleTheme);

async function simulate() {
    errorMessage.value = '';

    for (const [key, value] of Object.entries(formData)) {
        if (value === null || value === '') {
            showErrorPopup(`⚠️ Please fill in ${key.replace(/([A-Z])/g, ' $1')}.`);
            return;
        }
    }

    const endAge = 100;
    let totalAmount = formData.startingInvestment;
    let annualTargetExpense = formData.monthlyTargetExpense * 12;
    const annualInvestment = formData.monthlyInvestment * 12;
    resultRows.value = [];
    financialFreedomAge.value = null;

    for (let age = formData.currentAge; age <= endAge; age++) {
        const dividendIncome = totalAmount * (formData.dividendYield / 100);
        const assetGrowth = totalAmount * (formData.assetGrowthRate / 100);
        const endOfYearAmount =
            totalAmount +
            annualInvestment +
            formData.annualLumpSumInvestment +
            dividendIncome +
            assetGrowth;

        resultRows.value.push({
            age,
            startingInvestment: totalAmount,
            monthlyInvestment: formData.monthlyInvestment,
            endOfYearAmount,
            dividendIncome,
            annualTargetExpense,
        });

        if (financialFreedomAge.value === null && dividendIncome >= annualTargetExpense) {
            financialFreedomAge.value = age;
        }

        totalAmount = endOfYearAmount;
        annualTargetExpense *= 1 + formData.inflationRate / 100;
    }

    await nextTick();
    document.querySelector('.table-wrapper')?.scrollIntoView({ behavior: 'smooth' });
}

function formatKMB(num: number): string {
    if (num >= 1_000_000_000) {
        return (num / 1_000_000_000).toFixed(2).replace(/\.00$/, '') + 'B';
    } else if (num >= 1_000_000) {
        return (num / 1_000_000).toFixed(2).replace(/\.00$/, '') + 'M';
    } else if (num >= 1_000) {
        return (num / 1_000).toFixed(2).replace(/\.00$/, '') + 'K';
    } else {
        return num.toFixed(2);
    }
}
</script>

<template>
    <div class="container">
        <div class="theme-toggle-wrapper">
            <label class="switch">
                <input type="checkbox" v-model="isDark" />
                <span class="slider"></span>
            </label>
            <span class="theme-label">{{ isDark ? ' Dark Mode' : ' Light Mode' }}</span>
        </div>

        <transition name="fade">
            <div v-if="showError" class="popup-error">{{ errorMessage }}</div>
        </transition>

        <h1>Financial Freedom Simulator</h1>

        <form @submit.prevent="simulate">
            <div class="form-group" v-for="(label, id) in {
                currentAge: 'Current Age',
                startingInvestment: 'Starting Investment',
                monthlyInvestment: 'SIP (Monthly Investment)',
                annualLumpSumInvestment: 'LSI (Annual Lump Sum Investment)',
                dividendYield: 'Dividend Yield (%)',
                assetGrowthRate: 'Capital Appreciation Rate (%)',
                inflationRate: 'Inflation Rate (%)',
                monthlyTargetExpense: 'Target Monthly Passive Income'
            }" :key="id">
                <label :for="id" :class="{ 'highlight-label': id === 'monthlyTargetExpense' }">{{ label }}</label>
                <input :id="id" type="number" v-model.number="formData[id]" min="0" step="0.01"
                    :placeholder="placeholders[id]" />
            </div>

            <button type="submit">Simulate</button>
        </form>

        <div style="margin-top: 1rem">
            <template v-if="financialFreedomAge !== null">
                🎉 You reach financial freedom at age
                <strong class="freedom-age">{{ financialFreedomAge }}</strong>
            </template>
            <template v-else>
                🚀 Keep going! You're on your way to financial freedom!
            </template>
        </div>

        <div class="table-wrapper" v-if="resultRows.length > 0">
            <table>
                <thead>
                    <tr>
                        <th>Age</th>
                        <th>Starting Investment</th>
                        <th>End of Year Investment</th>
                        <th>Passive Income</th>
                        <th>Target Annual Passive Income</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="row in resultRows" :key="row.age"
                        :class="{ highlight: row.age === financialFreedomAge }">
                        <td>{{ row.age }}</td>
                        <td>{{ formatKMB(row.startingInvestment) }}</td>
                        <td>{{ formatKMB(row.endOfYearAmount) }}</td>
                        <td>{{ formatKMB(row.dividendIncome) }}</td>
                        <td>{{ formatKMB(row.annualTargetExpense) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.container {
    max-width: 960px;
    margin: 2rem auto;
    padding: 1rem 2rem;
    text-align: center;
}

.form-group {
    margin-bottom: 1rem;
    text-align: left;
    display: flex;
    flex-direction: column;
    align-items: center;
}

input {
    width: 300px;
    max-width: 100%;
    padding: 0.4rem;
    border-radius: 6px;
    border: 1px solid #999;
    background: var(--input-bg);
    color: var(--text-color);
}

label {
    margin-bottom: 0.3rem;
}

button {
    margin-top: 1rem;
    padding: 0.6rem 1.2rem;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

button:hover {
    background-color: #43a047;
}

/* ✅ 美化後的 switch */
.theme-switch {
    position: fixed;
    top: 1rem;
    right: 1rem;
    display: flex;
    align-items: center;
    gap: 0.4rem;
    padding: 0.3rem 0.8rem;
    background-color: var(--input-bg, #fff);
    color: var(--text-color);
    border-radius: 999px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
    z-index: 1000;
    font-size: 0.85rem;
    white-space: nowrap;
}

.switch {
    position: relative;
    display: inline-block;
    width: 42px;
    height: 22px;
    flex-shrink: 0;
}

.switch input {
    opacity: 0;
    width: 0;
    height: 0;
}

.slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    transition: 0.3s;
    border-radius: 22px;
}

.slider:before {
    position: absolute;
    content: "";
    height: 16px;
    width: 16px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: 0.3s;
    border-radius: 50%;
}

input:checked+.slider {
    background-color: #4caf50;
}

input:checked+.slider:before {
    transform: translateX(20px);
}

.highlight {
    background-color: #ffd700 !important;
    color: #222 !important;
    font-weight: bold;
    transition: background-color 0.3s ease, color 0.3s ease;
}

.highlight-label {
    color: #ff5722;
    font-weight: bold;
}

.freedom-age {
    color: #00e676;
    font-weight: bold;
}

.table-wrapper {
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    margin-top: 2rem;
}

thead {
    background-color: var(--thead-bg, #f0f0f0);
}

th,
td {
    padding: 0.75rem 1rem;
    text-align: center;
    border: 1px solid #ddd;
}

/* popup-style error */
.popup-error {
    position: fixed;
    top: 1rem;
    left: 50%;
    transform: translateX(-50%);
    background-color: #ffe6e6;
    color: #d32f2f;
    border: 1px solid #d32f2f;
    padding: 0.75rem 1.5rem;
    border-radius: 8px;
    z-index: 9999;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
    font-weight: bold;
}

/* dark mode error style */
[data-theme='dark'] .popup-error {
    background-color: #3a0f0f;
    color: #ff8a80;
    border-color: #ff8a80;
}

/* 動畫效果 */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.4s;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>