<script setup lang="ts">
import { ref, defineEmits, onMounted } from 'vue'

const props = defineProps<{ sizeNum: number | undefined }>()
const emit = defineEmits(['num-change'])

function handleInputChange(evt: Event) {
  // @ts-ignore
  emit('num-change', evt.target.value)
}

const initNum = props.sizeNum

onMounted(() => {
  console.log(initNum);
})
</script>

<template>
  <div class="flex gap-2">
    <p>S</p>
    <p>M</p>
    <p>L</p>
    <input :value="sizeNum"
      type="number" step="5" class="w-8"
      @input="handleInputChange"
    >
  </div>
</template>

<style scoped>
input[type=number] {
  -moz-appearance: textfield;
}

input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>
