<script setup lang="ts">
defineProps<{
  answer: {
    name: string
    imgHref: string
    description: string
    borderStyle: string
  }
  displayBorder: boolean
  displayForMobile?: boolean
  saveAnswers: (index: number, sectionIndex: number, selectionType: string) => void
}>()
</script>

<template>
  <v-img
    :src="answer.imgHref"
    :style="{ border: displayBorder && !displayForMobile ? '3px solid #7BE891' : '' }"
    class="answer-image"
    @click="saveAnswers"
  ></v-img>
</template>

<style scoped>
.answer-image {
  width: 100px;
  height: 100px;
  border-radius: 50%;
}
.answer-image:hover {
  cursor: pointer;
  transition: 0.5;
  transform: scale(1.1);
}
</style>
