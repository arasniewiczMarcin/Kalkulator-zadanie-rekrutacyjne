<script setup lang="ts">
import { ref } from 'vue'
import data from '../data/data.json'
import ImageAnswer from './ImageAnswer.vue'
import SummarySection from './SummarySection.vue'

const QUESTION_ONE_INDEX = 0
const SECTIONS_INDEX = 1
const ENDING_OPERATIONS_INDEX = 2
const VALUES_INDEX = 4

const sectionsData = data[SECTIONS_INDEX].items
const valuesData = data[VALUES_INDEX].items
const endingOperations = data[ENDING_OPERATIONS_INDEX].items

const numberOfAnswers = 15
const answers = ref(Array(numberOfAnswers).fill(false))
const summaryMessages = ref<string[]>([])

const displayForMobile = ref(window.innerWidth < 600)

const counting = ref({
  roller_multiplier: valuesData[0].defaultValue,
  excavator_crane_multiplier: valuesData[1].defaultValue,
  speed_multiplier: valuesData[2].defaultValue,
  workday_price: valuesData[3].defaultValue,
  multiplier: valuesData[4].defaultValue,
  building_days: valuesData[5].defaultValue,
  multiplied_building_days: valuesData[6].defaultValue,
  active_working_hours: valuesData[7].defaultValue,
  price: valuesData[8].defaultValue
})

const saveAnswers = (id: number, sectionIndex: number, selectionType: string) => {
  if (selectionType === 'single') {
    const index = sectionIndex * 3
    answers.value[index] = false
    answers.value[index + 1] = false
    answers.value[index + 2] = false
  }
  if (answers.value[id]) answers.value[id] = false
  else answers.value[id] = true
  createSummaryMessage()
}

const createSummaryMessage = () => {
  summaryMessages.value = []
  sectionsData.forEach((element, index) => {
    if (
      element.mustSelectRequirement?.enabled &&
      !answers.value[index * 3] &&
      !answers.value[index * 3 + 1] &&
      !answers.value[index * 3 + 2]
    ) {
      summaryMessages.value.push(element.mustSelectRequirement?.messageIfNotSelected)
    }
  })
  if (summaryMessages.value.length > 0) return
  let usedBuildingEquipment = []
  for (let i = 0; i < 3; i++) {
    if (answers.value[i]) {
      usedBuildingEquipment.push(sectionsData[0].items[i].name)
    }
  }
  summaryMessages.value.push(usedBuildingEquipment.join(' + '))
  SumUpCosts()
}

const SumUpCosts = () => {
  const answer_indexes = {
    excavator: 0,
    crane: 1,
    roller: 2
  }

  counting.value.roller_multiplier = valuesData[0].defaultValue
  counting.value.excavator_crane_multiplier = valuesData[1].defaultValue
  counting.value.speed_multiplier = valuesData[2].defaultValue
  counting.value.workday_price = valuesData[3].defaultValue
  counting.value.multiplier = valuesData[4].defaultValue
  counting.value.building_days = valuesData[5].defaultValue
  counting.value.multiplied_building_days = valuesData[6].defaultValue
  counting.value.active_working_hours = valuesData[7].defaultValue
  counting.value.price = valuesData[8].defaultValue

  //operations if enabled

  sectionsData[QUESTION_ONE_INDEX].items.forEach((answer) => {
    answer.operationsIfEnabled.forEach((operation) => {
      let answer_name: string = operation.executeIfScopeEnabled
      if (answers.value[answer_indexes[answer_name]]) {
        if (operation.type === 'add') {
          counting.value[operation.relatedValue] += operation.number
        } else if (operation.type === 'multiply') {
          counting.value[operation.relatedValue] *= operation.number
        }
      }
    })
  })
  sectionsData.forEach((section, sectionIndex) => {
    if (sectionIndex !== QUESTION_ONE_INDEX) {
      section.items.forEach((answer, answerIndex) => {
        if (answers.value[sectionIndex * 3 + answerIndex]) {
          answer.operationsIfEnabled.forEach((operation) => {
            if (operation.type === 'add') {
              counting.value[operation.relatedValue] += operation.number
            } else if (operation.type === 'multiply') {
              counting.value[operation.relatedValue] *= operation.number
            }
          })
        }
      })
    }
  })

  //ending operations

  endingOperations.forEach((operation) => {
    if (operation.type === 'addValues') {
      counting.value[operation.toValue] += counting.value[operation.fromValue]
    } else if (operation.type === 'copyValues') {
      counting.value[operation.toValue] = counting.value[operation.fromValue]
    } else if (operation.type === 'multiplyValues') {
      counting.value[operation.value] *= counting.value[operation.byValue]
    } else if (operation.type === 'multiply') {
      counting.value[operation.relatedValue] *= operation.number
    }
  })
}
</script>

<template>
  <v-sheet
    class="section-container"
    :class="{ 'section-container-mobile': displayForMobile }"
    v-for="(section, sectionIndex) in sectionsData"
    :key="section.name"
  >
    <div class="text-h5 section-center">{{ section.name }}</div>
    <v-row class="section-center">
      <v-col
        v-for="(answer, index) in section.items"
        :key="answer.name"
        cols="12"
        sm="6"
        md="4"
        lg="3"
      >
        <v-tooltip v-if="!displayForMobile" :text="answer.description" arrow location="bottom">
          <template v-slot:activator="{ props }">
            <div class="answer answer-name" v-bind="props">
              <div class="text-h6 answer-name">{{ answer.name }}</div>
              <ImageAnswer
                :answer="answer"
                :displayBorder="answers[sectionIndex * 3 + index]"
                :saveAnswers="
                  () => saveAnswers(sectionIndex * 3 + index, sectionIndex, section.selectionType)
                "
              />
            </div>
          </template>
        </v-tooltip>
        <div v-else>
          <div
            class="answer-mobile"
            :class="{ 'selected-answer-mobile': answers[sectionIndex * 3 + index] }"
          >
            <ImageAnswer
              class="image-mobile"
              :answer="answer"
              :displayBorder="answers[sectionIndex * 3 + index]"
              :displayForMobile="displayForMobile"
              :saveAnswers="
                () => saveAnswers(sectionIndex * 3 + index, sectionIndex, section.selectionType)
              "
            />

            <div class="text-mobile">
              <div class="text-h6 answer-name">{{ answer.name }}</div>
              <div class="text-h6 description-mobile">{{ answer.description }}</div>
            </div>
          </div>
        </div>
      </v-col>
    </v-row>
  </v-sheet>
  <SummarySection
    :summaryMessages="summaryMessages"
    :workHours="counting.active_working_hours"
    :price="counting.price"
    :displayForMobile="displayForMobile"
  />
</template>

<style scoped>
/* Colors */
.answer-name {
  color: black;
}

/* Section Container Styles */
.section-container {
  margin: 2rem auto;
  background-color: red;
  padding: 1rem;
  width: 60%;
}

.section-center {
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  margin: 0.5rem 0.5rem;
}
.answer {
  margin: 1rem 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.answer-mobile {
  display: flex;
  background-color: white;
  align-items: center;
  justify-content: center;
  margin: 0;
}

.text-mobile {
  margin: 0 1rem;
}
.description-mobile {
  color: gray;
  font-size: 5px;
}

.selected-answer-mobile {
  background-color: #7be891;
}
.section-container-mobile {
  width: 95%;
}
.image-mobile {
  margin: 0.5rem 0.5rem;
}
</style>
