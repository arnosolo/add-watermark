<script setup lang="ts">
import { inject, reactive, ref } from 'vue'
import ProcedureBar from "../components/ProcedureBar.vue";
import ImagePicker from "../components/ImagePicker.vue";
import WatermarkEditor from "../components/WatermarkEditor.vue";
import WatermarkDeployer from "../components/WatermarkDeployer.vue";
import {WatermarkProcedure} from '../types/WatermarkProcedure'

let procedureState = ref(WatermarkProcedure.pickImages)
let procedureLen = 0
for(let p in WatermarkProcedure) {
  procedureLen ++
}
procedureLen /=2
const setProcedureState = (newState: WatermarkProcedure) => {
  if(newState >=1 && newState <= procedureLen) {
    procedureState.value = newState
  } else {
    console.warn("New procedureState outrange, ignore")
  }
}
const nextProcedure = () => {
  setProcedureState(procedureState.value + 1)
}
const prevProcedure = () => {
  setProcedureState(procedureState.value - 1)
}

let selectedFiles = inject("selectedFiles", reactive([])) as Array<File>
let setSelectedFile = inject("setSelectedFile", () => {}) as (newFiles: Array<File>) => void


</script>

<template>
  <div class="">
    <!-- <ProcedureBar :currentState="procedureState" :procedureLen="procedureLen"></ProcedureBar> -->
    <!-- <ImagePicker :setSelectedFile="setSelectedFile" :selectedFiles="selectedFiles" :nextProcedure="nextProcedure"></ImagePicker> -->
    <!-- <WatermarkEditor :selectedFiles="selectedFiles" :prevProcedure="prevProcedure" :nextProcedure="nextProcedure" v-show="procedureState == WatermarkProcedure.createWatermark"></WatermarkEditor> -->
    <WatermarkDeployer  :setSelectedFile="setSelectedFile" :selectedFiles="selectedFiles" :prevProcedure="prevProcedure" :nextProcedure="nextProcedure"></WatermarkDeployer>
  </div>
</template>

<style scoped>

</style>
