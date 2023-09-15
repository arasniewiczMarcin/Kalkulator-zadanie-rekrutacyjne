<script setup lang="ts">
import { ref } from 'vue'
import data from '../data/data.json'

defineProps<{
  summaryMessages: string[]
  workHours: number
  price: number
  displayForMobile: boolean
}>()

const SUMMARY_INDEX = 3
const summaryData = ref(data[SUMMARY_INDEX])
</script>
<template>
  <v-sheet class="summary-container" :class="{ 'summary-mobile': displayForMobile }">
    <div
      class="summary-message"
      v-if="summaryMessages[0] !== 'Question 1 - Choose road building equipment'"
    >
      <div class="text-h5" v-for="message in summaryMessages" :key="message">
        {{ message }}
      </div>
    </div>
    <div class="summary-message" v-else>
      <div class="text-h5">
        {{ summaryData.noScopeDisclaimer }}
      </div>
    </div>

    <div>
      <v-divider :thickness="2" class="border-opacity-75" color="white" />
    </div>
    <div class="text-h7">
      {{ summaryData.items[0].text + (workHours ? ' ' + workHours.toFixed(2) : workHours) }}
    </div>
    <div class="text-h4">
      {{ summaryData.totalPriceDescription + (price ? ' ' + price.toFixed() : ' -') }}
    </div>
  </v-sheet>
</template>

<style scoped>
.summary-message {
  padding-bottom: 1rem;
  color: black;
}
.summary-container {
  margin: 2rem auto;
  background-color: red;
  padding: 1rem;
  width: 60%;
  color: white;
}
.summary-mobile {
  width: 95%;
}
</style>
