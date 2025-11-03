<template>
  <div class="w-full">
    <label class="block text-gray-600 mb-2">{{ label }}</label>
    <select
      v-model="selectedValue"
      class="w-full border border-gray-300 rounded-lg p-2 focus:ring focus:ring-blue-200 focus:outline-none"
    >
      <option v-for="cur in currencies" :key="cur" :value="cur">
        {{ cur }}
      </option>
    </select>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  label: String,
  currencies: Array,
  modelValue: String,
});

const emit = defineEmits(["update:modelValue"]);

const selectedValue = computed({
  get: () => props.modelValue,
  set: (val) => emit("update:modelValue", val),
});
</script>
