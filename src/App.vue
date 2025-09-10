<script setup>

import {computed, onMounted, ref} from "vue";
import Button from "./components/Button.vue";

let user = ref(null);

let isOnTime = ref(true);

let fastEnough = ref(areYouFastEnough());

let restOfDay = msToMidnight();

let timer = ref(convertMsToTime());

let counter = ref(0);


function areYouFastEnough(){

  console.log("YouFast Enough");
  if(user.value) {
    console.log("are you fast? bool", Date.now() - user.value.timer > 86400000);
    return Date.now() - user.value.timer > 86400000
  }
}

console.log("restOfDay",restOfDay);
let interval = null;

function msToMidnight(){

  let oneDay = 86400000
  let msIntoDay = Date.now() % oneDay;

  console.log("msintoDAy:",msIntoDay);
  console.log("oneday:", oneDay);
  console.log("tid igjen til midnatt:", oneDay - msIntoDay);

  return oneDay - msIntoDay;
}



function amIOnStreak(){

  //holde streaken
  let streakTimeWindow = user.value.msToMidnight + 86400000;
  // if streakTimeWindow
  // counter ++
  //else if !streakTimeWindow
  // counter = 0

  //if counter >== 2
  //user.value.streak++

  // if(new Date().getMilliseconds() > streakTimeWindow) return false;
}



function convertMsToTime(){

  let oneDay = 86400000
  let msIntoDay = Date.now() % oneDay;

  let intoDay = oneDay - msIntoDay;

  const totalSeconds = Math.floor(intoDay / 1000);

  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;

  console.log(`${hours}:${minutes}:${seconds}`);
  return {hours, minutes, seconds};
}

onMounted(async () => {

  let req = await fetch("http://localhost:4002/user")
  user.value = await req.json()
  console.log(user.value)

  areYouFastEnough();

  startTimer();
  convertMsToTime()

  msToMidnight()
  if(user.value){getNextAvailableTime()}


})

async function updateScore(){
  console.log("update score")
  if(!user.value){
    return;
  }
  amIOnStreak();
  let newScore = user.value.score + 1
  let newStreak = user.value.streak + 1
  let newTime = Date.now();
  let toMidnight = msToMidnight();

  let req = await fetch("http://localhost:4002/user", {
    method: "PUT",
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
      ...user.value,
      score: isOnTime.value ? newScore : user.value.score,
      streak: isOnTime.value ? newStreak : 0,
      timer: newTime,
      msToMidnight: toMidnight,
      counter: counter.value
    })
  })

  user.value = await req.json();
}


function eachTick(){
// console.log("eachTick")

  timer.value = convertMsToTime();
}

function startTimer(){
interval = setInterval(eachTick, 1000)
}

function stopTimer(){
clearInterval(interval)
  interval = null;
}

function getNextAvailableTime(){
  const lastDate = new Date(user.value.timer);
  const nextDate = new Date(lastDate);
  nextDate.setDate(lastDate.getDate() + 1);
  nextDate.setHours(0, 0, 0, 0);

  console.log("last date:",lastDate);
  console.log("next date:",nextDate);

  return nextDate;
}

</script>

<template>

  <div class="container">
    <h1>Habit tracker :)</h1>
    <div class="user-stats" v-if="user">
      Score: {{user.score ?? 0}} Streak: {{user.streak}}


    </div>
    <button  @click="updateScore">Check!</button>
    <p>Timer:</p>
    <strong>{{timer.hours}} Hours - {{timer.minutes}} Minutes - {{timer.seconds}} Seconds</strong>
    <button @click="stopTimer">Stop timer</button>
  </div>
-
</template>

<style scoped>

</style>
