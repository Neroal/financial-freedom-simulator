<script setup lang="ts">
import { ref, watchEffect, onMounted, reactive, nextTick, onUpdated } from 'vue';
import {
    Chart,
    LineController,
    LineElement,
    PointElement,
    LinearScale,
    BarController,
    BarElement,
    Title,
    CategoryScale,
    Tooltip,
    Legend,
} from 'chart.js';

Chart.register(
    LineController,
    LineElement,
    PointElement,
    BarController,
    BarElement,
    LinearScale,
    Title,
    CategoryScale,
    Tooltip,
    Legend
);

let chartInstance: Chart | null = null;

onUpdated(() => {
    if (resultRows.value.length === 0) return;

    // 新增 targetData
    const targetData = resultRows.value.map((r) => r.monthlyDividendTarget);

    const ctx = document.getElementById('result-chart') as HTMLCanvasElement;
    if (!ctx) return;

    const labels = resultRows.value.map((r) => r.age);
    const dividendData = resultRows.value.map((r) => r.monthlyDividend);

    if (chartInstance) {
        chartInstance.destroy();
    }

    chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
            labels,
            datasets: [
                {
                    type: 'line',
                    label: 'Monthly Dividend Goal',
                    data: targetData,
                    borderColor: 'rgba(255, 99, 132, 1)',
                    backgroundColor: 'rgba(255, 99, 132, 0.2)',
                    borderDash: [5, 5],
                    tension: 0.3,
                    yAxisID: 'y',
                },
                {
                    type: 'bar',
                    label: 'Monthly Dividend',
                    data: dividendData,
                    backgroundColor: 'rgba(75, 192, 192, 0.5)',
                    borderColor: 'rgba(75, 192, 192, 1)',
                    borderWidth: 1,
                    yAxisID: 'y',
                },
            ],
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: {
                    labels: {
                        color: isDark.value ? '#fff' : '#000',
                        font: {
                            size: 12,
                            weight: 'bold',
                        },
                    },
                },
                tooltip: {
                    backgroundColor: isDark.value ? '#333' : '#fff',
                    titleColor: isDark.value ? '#fff' : '#000',
                    bodyColor: isDark.value ? '#fff' : '#000',
                },
            },
            scales: {
                x: {
                    ticks: {
                        color: isDark.value ? '#ccc' : '#333',
                    },
                    grid: {
                        color: isDark.value ? 'rgba(255, 255, 255, 0.05)' : 'rgba(0, 0, 0, 0.05)',
                    },
                },
                y: {
                    beginAtZero: true,
                    position: 'left',
                    title: {
                        display: true,
                        text: 'Dividend',
                        color: isDark.value ? '#ccc' : '#333',
                    },
                    ticks: {
                        color: isDark.value ? '#ccc' : '#333',
                    },
                    grid: {
                        color: isDark.value ? 'rgba(255, 255, 255, 0.05)' : 'rgba(0, 0, 0, 0.05)',
                    },
                },
            },
        },
    });
});

const isDark = ref(true);

const placeholders = {
    currentAge: 18,
    startingInvestment: 0,
    monthlyInvestment: 1000,
    annualLumpSumInvestment: 128000,
    dividendYield: 5,
    inflationRate: 2,
    monthlyDividendTarget: 100000,
};

const formData = reactive({
    currentAge: 18,
    startingInvestment: 0,
    monthlyInvestment: 1000,
    annualLumpSumInvestment: 128000,
    dividendYield: 5,
    inflationRate: 2,
    monthlyDividendTarget: 100000,
});

const resultRows = ref<
    {
        age: number;
        totalAssets: number;
        monthlyDividend: number;
        monthlyDividendTarget: number;
    }[]
>([]);

const financialFreedomAge = ref<number | null>(null);
const errorMessage = ref('');

function toggleTheme() {
    document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light');
}

onMounted(() => {
    toggleTheme();
});

watchEffect(toggleTheme);

async function simulate() {
    errorMessage.value = '';

    const endAge = 100;
    let totalAssets = formData.startingInvestment;
    let monthlyDividendTarget = formData.monthlyDividendTarget;
    const annualInvestment = formData.monthlyInvestment * 12 + formData.annualLumpSumInvestment;
    resultRows.value = [];
    financialFreedomAge.value = null;

    for (let age = formData.currentAge; age <= endAge; age++) {
        const annualDividend = totalAssets * (formData.dividendYield / 100);
        const endOfYearAssets = totalAssets + annualInvestment + annualDividend;
        const monthlyDividend = annualDividend / 12;

        resultRows.value.push({
            age,
            totalAssets,
            monthlyDividend,
            monthlyDividendTarget,
        });

        if (financialFreedomAge.value === null && monthlyDividend >= monthlyDividendTarget) {
            financialFreedomAge.value = age;
        }

        totalAssets = endOfYearAssets;
        monthlyDividendTarget *= 1 + formData.inflationRate / 100;
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

        <h1>Financial Freedom Simulator</h1>

        <form @submit.prevent="simulate">
            <div
                class="form-group"
                v-for="(label, id) in {
                    currentAge: 'Current Age',
                    startingInvestment: 'Starting Investment',
                    monthlyInvestment: 'SIP (Monthly Investment)',
                    annualLumpSumInvestment: 'LSI (Annual Lump Sum Investment)',
                    dividendYield: 'Dividend Yield (%)',
                    inflationRate: 'Inflation Rate (%)',
                    monthlyDividendTarget: 'Monthly Dividend Goal',
                }"
                :key="id"
            >
                <label :for="id" :class="{ 'highlight-label': id === 'monthlyDividendTarget' }">{{
                    label
                }}</label>
                <input
                    :id="id"
                    type="number"
                    v-model.number="formData[id]"
                    min="0"
                    step="0.01"
                    :placeholder="String(placeholders[id])"
                    required
                />
            </div>

            <button type="submit">Simulate</button>
        </form>

        <div style="margin-top: 1rem">
            <template v-if="financialFreedomAge !== null">
                🎉 You reach financial freedom at age
                <strong class="freedom-age">{{ financialFreedomAge }}</strong>
            </template>
            <template v-else> 🚀 Keep going! You're on your way to financial freedom! </template>
        </div>

        <div class="table-wrapper" v-if="resultRows.length > 0">
            <div
                style="margin-top: 2rem; position: relative; height: 400px"
                v-if="resultRows.length > 0"
            >
                <canvas id="result-chart"></canvas>
            </div>

            <table>
                <thead>
                    <tr>
                        <th>Age</th>
                        <th class="desktop-only">Total Assets</th>
                        <th>Monthly Dividend</th>
                        <th>Dividend Goal</th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="row in resultRows"
                        :key="row.age"
                        :class="{ highlight: row.age === financialFreedomAge }"
                    >
                        <td>{{ row.age }}</td>
                        <td class="desktop-only">{{ formatKMB(row.totalAssets) }}</td>
                        <td>{{ formatKMB(row.monthlyDividend) }}</td>
                        <td>{{ formatKMB(row.monthlyDividendTarget) }}</td>
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
    content: '';
    height: 16px;
    width: 16px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: 0.3s;
    border-radius: 50%;
}

input:checked + .slider {
    background-color: #4caf50;
}

input:checked + .slider:before {
    transform: translateX(20px);
}

.highlight {
    background-color: #ffd700 !important;
    color: #222 !important;
    font-weight: bold;
    transition:
        background-color 0.3s ease,
        color 0.3s ease;
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

@media (max-width: 600px) {
    .desktop-only {
        display: none;
    }
}

canvas {
    max-width: 100%;
}
</style>
