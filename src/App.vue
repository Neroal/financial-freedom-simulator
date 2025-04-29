<script setup lang="ts">
import { ref } from 'vue';

const currentAge = ref<number | null>(null);
const startingAmount = ref<number | null>(null);
const monthlyInvestment = ref<number | null>(null);
const annualLumpSumInvestment = ref<number | null>(null);
const dividendYield = ref<number | null>(null);
const assetGrowthRate = ref<number | null>(null);
const inflationRate = ref<number | null>(null);
const monthlyTargetExpense = ref<number | null>(null);

const resultRows = ref<
    {
        age: number;
        startingAmount: number;
        monthlyInvestment: number;
        endOfYearAmount: number;
        dividendIncome: number;
        annualTargetExpense: number;
    }[]
>([]);

const financialFreedomAge = ref<number | null>(null);

function simulate() {
    if (
        currentAge.value === null ||
        startingAmount.value === null ||
        monthlyInvestment.value === null ||
        annualLumpSumInvestment.value === null ||
        dividendYield.value === null ||
        assetGrowthRate.value === null ||
        inflationRate.value === null ||
        monthlyTargetExpense.value === null
    ) {
        resultRows.value = [];
        financialFreedomAge.value = null;
        return;
    }

    const endAge = 100;
    let totalAmount = startingAmount.value;
    let annualTargetExpense = monthlyTargetExpense.value * 12;
    const annualInvestment = monthlyInvestment.value * 12;
    resultRows.value = [];
    financialFreedomAge.value = null;

    for (let age = currentAge.value; age <= endAge; age++) {
        const dividendIncome = totalAmount * (dividendYield.value / 100);
        const assetGrowth = totalAmount * (assetGrowthRate.value / 100);
        const endOfYearAmount =
            totalAmount +
            annualInvestment +
            annualLumpSumInvestment.value +
            dividendIncome +
            assetGrowth;

        resultRows.value.push({
            age,
            startingAmount: totalAmount,
            monthlyInvestment: monthlyInvestment.value,
            endOfYearAmount,
            dividendIncome,
            annualTargetExpense,
        });

        // 偵測財富自由
        if (financialFreedomAge.value === null && dividendIncome >= annualTargetExpense) {
            financialFreedomAge.value = age;
        }

        totalAmount = endOfYearAmount;
        annualTargetExpense *= 1 + inflationRate.value / 100;
    }
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
    <div>
        <h1>Financial Freedom Simulator</h1>
        <form @submit.prevent="simulate">
            <div>
                <label for="currentAge">Current Age:</label>
                <input id="currentAge" type="number" v-model.number="currentAge" min="0" />
            </div>
            <div>
                <label for="startingAmount">Starting Investment:</label>
                <input id="startingAmount" type="number" v-model.number="startingAmount" min="0" />
            </div>
            <div>
                <label for="monthlyInvestment">SIP (Monthly Investment):</label>
                <input
                    id="monthlyInvestment"
                    type="number"
                    v-model.number="monthlyInvestment"
                    min="0"
                />
            </div>
            <div>
                <label for="annualLumpSumInvestment">LSI (Annual Lump Sum Investment):</label>
                <input
                    id="annualLumpSumInvestment"
                    type="number"
                    v-model.number="annualLumpSumInvestment"
                    min="0"
                />
            </div>
            <div>
                <label for="dividendYield">Dividend Yield (%):</label>
                <input
                    id="dividendYield"
                    type="number"
                    v-model.number="dividendYield"
                    min="0"
                    step="0.01"
                />
            </div>
            <div>
                <label for="assetGrowthRate">Asset Growth Rate (%):</label>
                <input
                    id="assetGrowthRate"
                    type="number"
                    v-model.number="assetGrowthRate"
                    min="0"
                    step="0.01"
                />
            </div>
            <div>
                <label for="inflationRate">Inflation Rate (%):</label>
                <input
                    id="inflationRate"
                    type="number"
                    v-model.number="inflationRate"
                    min="0"
                    step="0.01"
                />
            </div>
            <div>
                <label for="monthlyTargetExpense">Target Monthly Passive Income:</label>
                <input
                    id="monthlyTargetExpense"
                    type="number"
                    v-model.number="monthlyTargetExpense"
                    min="0"
                />
            </div>
            <div>
                <button type="submit">Simulate</button>
            </div>
        </form>

        <div v-if="financialFreedomAge !== null" style="margin-top: 1rem">
            🎉 You reach financial freedom at age <strong>{{ financialFreedomAge }}</strong>
        </div>

        <table
            v-if="resultRows.length > 0"
            border="1"
            cellpadding="8"
            cellspacing="0"
            style="margin-top: 1rem"
        >
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
                <tr v-for="row in resultRows" :key="row.age">
                    <td>{{ row.age }}</td>
                    <td>{{ formatKMB(row.startingAmount) }}</td>
                    <td>{{ formatKMB(row.endOfYearAmount) }}</td>
                    <td>{{ formatKMB(row.dividendIncome) }}</td>
                    <td>{{ formatKMB(row.annualTargetExpense) }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<style scoped>
form {
    max-width: 400px;
    margin: 0 auto;
    padding: 1rem;
    background-color: #1e1e1e;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.05);
}

form > div {
    margin-bottom: 0.8rem;
    display: flex;
    flex-direction: column;
}

label {
    margin-bottom: 0.3rem;
    font-weight: 600;
    color: #fff;
}

input {
    padding: 0.5rem;
    border: 1px solid #555;
    border-radius: 4px;
    background-color: #2c2c2c;
    color: #fff;
}

button {
    padding: 0.6rem 1.2rem;
    background-color: #4caf50;
    color: white;
    font-weight: bold;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}
button:hover {
    background-color: #43a047;
}

h1 {
    text-align: center;
    font-size: 2rem;
    margin-bottom: 1rem;
    color: #fff;
}

table {
    width: 90%;
    margin: 1rem auto;
    border-collapse: collapse;
    color: #ddd;
    background-color: #1e1e1e;
}

th,
td {
    padding: 0.6rem 1rem;
    border: 1px solid #444;
    text-align: center;
}

thead {
    background-color: #333;
    font-weight: bold;
}

tbody tr:nth-child(odd) {
    background-color: #2a2a2a;
}

tbody tr:nth-child(even) {
    background-color: #1c1c1c;
}

strong {
    color: #00e676;
    font-weight: bold;
}
</style>
