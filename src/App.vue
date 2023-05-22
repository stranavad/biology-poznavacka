<script setup lang="ts">
import jsonData from './data.json';
import {Answer, Plant} from "./types";
import {ref} from "vue";
import {getRandomElements, shuffle} from "./utils.ts";
const data: Plant[] = jsonData.plants;

const currentPlant = ref<Plant | null>(null);
const answers = ref<Answer[]>([]);
const score = ref(0);

function answerPlant(answer: Answer){
  if(answer.name === currentPlant.value!.name){
    answers.value[answers.value.findIndex(({name}) => name === answer.name)].correct = true;
    setTimeout(() => {
      // Wrong answer count
      const wrongAnswers = answers.value.filter(({wrong}) => wrong).length;

      score.value += 5 - wrongAnswers;
      getNextPlant();
    }, 750);
    return;
  }

  answers.value[answers.value.findIndex(({name}) => name === answer.name)].wrong = true;
}

function getNextPlant (){
  currentPlant.value = data[Math.floor(Math.random() * data.length)];
  answers.value = shuffle([...getRandomElements(data, 5).map(({name}) => name), currentPlant.value!.name]).map((name) => ({name, wrong: false, correct: false}))
}
function getImageUrl(image: string){
  return new URL(`./assets/plants/${image}`, import.meta.url).href
}

function getAnswerClass(answer: Answer){
  if(answer.wrong){
    return 'text-red-500 cursor-not-allowed bg-red-900/50 pointer-events-none';
  }

  if(answer.correct){
    return 'text-green-500 cursor-not-allowed bg-green-900/50 pointer-events-none'
  }

  return 'text-gray-300 dark:bg-gray-800 bg-gray-200 hover:opacity-70 '
}

function created(){
  getNextPlant();
}

created();
</script>

<template>
  <div class="max-w-xl mx-auto mt-2 px-3">
    <div class="flex justify-end mb-3">
      <span class="text-lg font-semibold text-gray-300">Skore: <span class="text-2xl">{{score}}</span></span>
    </div>
    <div v-if="currentPlant" class="my-2">
      <img :src="getImageUrl(currentPlant.image)" alt="Plant" style="aspect-ratio: 576/392;" class="w-full object-contain"/>
    </div>
    <div class="flex flex-col">
      <button
          v-for="answer in answers"
          class="block text-left text-xs font-semibold py-3 px-2 rounded-md my-2 transition-opacity duration-100"
          :class="getAnswerClass(answer)"
          :disabled="answers.some(({correct}) => correct)"
          @click="answerPlant(answer)"
      >
        {{answer.name}}
      </button>
    </div>
    <div class="fixed bottom-1 left-0 right-0 flex justify-center  mt-20">
      <span class="text-xs dark:text-gray-400 text-gray-600">
        2023 | <a class="dark:text-gray-300 text-gray-700 font-semibold hover:dark:text-gray-200 hover:text-gray-800 transition-colors duration-100" href="https://www.linkedin.com/in/stranavadavid/">Gandalf The Gray</a>
      </span>
    </div>
  </div>
</template>

<style scoped>
</style>
