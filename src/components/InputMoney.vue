<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  modelValue: number
}>()

const emit = defineEmits<{
  'update:modelValue': [value: number]
}>()

const inputValue = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})
</script>

<template>
  <div class="input-content">
    <label class="label">Quantidade de Lules</label>
    <div class="input-wrapper">
      <span class="currency">R$</span>
      <input
        type="number"
        v-model.number="inputValue"
        min="0"
        step="0.01"
        class="input"
        placeholder="0.00"
      />
    </div>
  </div>
</template>

<style scoped>
.input-content {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.label {
  font-weight: bold;
  color: #666;
  text-align: left;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.currency {
  position: absolute;
  left: 12px;
  color: #666;
  font-weight: bold;
}

.input {
  width: 100%;
  padding: 0.5rem;
  padding-left: 2.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s;
  background-color: darksalmon;
}

.input:focus {
  border-color: #646cff;
}

/* Remove spinner buttons from number input */
.input::-webkit-outer-spin-button,
.input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.input[type=number] {
  -moz-appearance: textfield;
}
</style>
