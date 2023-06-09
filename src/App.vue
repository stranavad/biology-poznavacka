<script setup lang="ts">
import jsonData from './data.json';
import {Answer, Plant} from "./types";
import {ref} from "vue";
import {getRandomElements, shuffle} from "./utils.ts";
const data: Plant[] = jsonData.plants;

const nextImage = ref<string | null>(null);
const currentPlant = ref<Plant | null>(null);
const answers = ref<Answer[]>([]);
const score = ref(0);

function updateScore(value: number){
  score.value = value;

  localStorage.setItem('score', value.toString());
}

function answerPlant(answer: Answer){
  if(answer.name === currentPlant.value!.name){
    answers.value[answers.value.findIndex(({name}) => name === answer.name)].correct = true;

    // Wrong answer count
    const wrongAnswers = answers.value.filter(({wrong}) => wrong).length;
    let scoreDelta = 10;

    if(wrongAnswers){
      scoreDelta = -2 * wrongAnswers
    }

    if(score.value + scoreDelta > 0){
      updateScore(score.value + scoreDelta)
    }

    getNextPlant();
    return;
  }

  answers.value[answers.value.findIndex(({name}) => name === answer.name)].wrong = true;
}

function getNextPlant (firstPlant:boolean=false){

  let newPlant = data[Math.floor(Math.random() * data.length)];
  let newAnswers = shuffle([...getRandomElements(data.filter(({name}) => name !== newPlant!.name), 5).map(({name}) => name), newPlant.name]).map((name) => ({name, wrong: false, correct: false}));

  if(!firstPlant){
    nextImage.value = newPlant.image;
  }

  setTimeout(() => {
    currentPlant.value = newPlant;
    answers.value = newAnswers
  }, firstPlant ? 0 : 750);
}
function getImageUrl(image: string){
  return new URL(`./assets/plants/${image}`, import.meta.url).href
}

function getAnswerClass(answer: Answer){
  if(answer.wrong){
    return 'text-red-500 cursor-not-allowed dark:bg-red-900/50 bg-red-200/50 pointer-events-none';
  }

  if(answer.correct){
    return 'text-green-500 cursor-not-allowed dark:bg-green-900/50 bg-green-200/50 pointer-events-none'
  }

  return 'dark:text-gray-300 text-gray-700 dark:bg-gray-800 bg-gray-300 hover:opacity-70 '
}

function created(){
  // Check if we have score in LS
  const lsScore = Number(localStorage.getItem('score'));

  // Set score from LS
  if(!isNaN(lsScore)){
    score.value = lsScore
  }

  getNextPlant(true);
}

created();
</script>

<template>
  <div class="max-w-xl mx-auto mt-2 px-3">
    <img v-if="nextImage" :src="getImageUrl(nextImage)" style="display: none">
    <div class="flex justify-end mb-2">
      <span class="text-lg font-semibold dark:text-gray-300 text-gray-800">Skore: <span class="text-2xl">{{score}}</span></span>
    </div>
    <div v-if="currentPlant" class="my-2">
      <img :src="getImageUrl(currentPlant.image)" alt="Plant" style="aspect-ratio: 576/392;" class="w-full object-contain"/>
    </div>
    <div class="flex flex-col">
      <button
          v-for="answer in answers"
          class="block text-left text-xs font-semibold py-3 px-2 rounded-md my-2 transition-opacity duration-100"
          :key="`${currentPlant!.name}-${answer.name}`"
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
