<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  selectedFiles: Array<File>,
  nextProcedure: () => void,
  prevProcedure: () => void,
}>()

const toNextStep = () => {
  props.nextProcedure()
}

const toPrevStep = () => {
  props.prevProcedure()
}
</script>

<template>
  <div class="flex">
    <div class="flex">
      <h1>WatermarkEditor</h1>
      <button @click="toPrevStep">上一步</button>
      <button @click="toNextStep">下一步</button>
    </div>
    <div class="flex">
      
    </div>
  </div>
</template>

<style scoped>

</style>
