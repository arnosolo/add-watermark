<script setup lang="ts">
import { ref } from 'vue'

defineProps<{ currentState: number, procedureLen: number }>()
</script>

<template>
  <h1>Step {{currentState}} / {{procedureLen}}</h1>
</template>

<style scoped>

</style>
