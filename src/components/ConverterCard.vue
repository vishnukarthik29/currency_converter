<template>
  <div class="bg-white p-6 rounded-2xl shadow-lg w-full max-w-md">
    <h1 class="text-2xl font-bold text-center mb-4">Currency Converter</h1>

    <!-- Amount Input -->
    <div class="mb-4">
      <label class="block text-gray-600 mb-2">Amount</label>
      <input
        v-model.number="amount"
        type="number"
        placeholder="Enter amount"
        class="w-full border border-gray-300 rounded-lg p-2 focus:ring focus:ring-blue-200 focus:outline-none"
      />
    </div>

    <!-- Currency Selection -->
    <div class="flex items-center justify-between gap-4 mb-4">
      <CurrencyDropdown
        v-model="fromCurrency"
        :currencies="currencies"
        label="From"
      />
      <button
        @click="swapCurrencies"
        class="bg-blue-500 text-white rounded-full p-2 hover:bg-blue-600 transition"
        title="Swap currencies"
      >
        ⇄
      </button>
      <CurrencyDropdown
        v-model="toCurrency"
        :currencies="currencies"
        label="To"
      />
    </div>

    <!-- Result -->
    <transition name="fade">
      <div v-if="convertedAmount !== null" class="mt-4 text-center">
        <p class="text-gray-600 text-sm">Converted Amount:</p>
        <p class="text-2xl font-semibold text-blue-600">
          {{ convertedAmount.toFixed(2) }} {{ toCurrency }}
        </p>
      </div>
    </transition>

    <!-- Loading & Error -->
    <div v-if="loading" class="text-center text-gray-500 mt-2">Loading...</div>
    <div v-if="error" class="text-center text-red-500 mt-2">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from "vue";
import axios from "axios";
import CurrencyDropdown from "./CurrencyDropdown.vue";

const amount = ref(1);
const fromCurrency = ref("USD");
const toCurrency = ref("INR");
const currencies = ref([]);
const convertedAmount = ref(null);
const loading = ref(false);
const error = ref(null);

const fetchRates = async () => {
  try {
    loading.value = true;
    error.value = null;

    const res = await axios.get(
      `https://open.er-api.com/v6/latest/${fromCurrency.value}`
    );

    if (res.data && res.data.result === "success") {
      const rate = res.data.rates[toCurrency.value];
      convertedAmount.value = amount.value * rate;
    } else {
      throw new Error("Invalid API response");
    }
  } catch (err) {
    console.error(err);
    error.value = "Failed to fetch exchange rates.";
  } finally {
    loading.value = false;
  }
};

const fetchCurrencies = async () => {
  try {
    const res = await axios.get("https://open.er-api.com/v6/latest/USD");
    if (res.data && res.data.result === "success") {
      currencies.value = Object.keys(res.data.rates);
    } else {
      throw new Error("Invalid currency data");
    }
  } catch (err) {
    console.error(err);
    error.value = "Failed to load currencies.";
  }
};

const swapCurrencies = () => {
  const temp = fromCurrency.value;
  fromCurrency.value = toCurrency.value;
  toCurrency.value = temp;
};

watch([amount, fromCurrency, toCurrency], fetchRates);

onMounted(async () => {
  await fetchCurrencies();
  await fetchRates();
});
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
