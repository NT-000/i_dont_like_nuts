<script setup>

import {computed, onMounted, ref} from "vue";
import '@fortawesome/fontawesome-free/css/all.css';
import Modal from "./components/Modal.vue";
import tjell from "/tjell.png"

let user = ref(null);
let isOpen = ref(false);

let statusMessage = ref("");

let milestones = ref([{id: 1, title: "10 day streak", img: "/public/10_day_streak.png"}, {
  id: 2,
  title: "20 days streak",
  img: "20.png"
}, {id: 3, title: "30 days streak", img: "/public/30_days.png"}]);

let now = ref(Date.now());
setInterval(() => {
  now.value = Date.now()
}, 1000);

let isAlreadyClickedToday = computed(() => {
  if (!user.value) return
  return now.value < (user.value.timer + user.value.msToMidnight);
})


let restOfDay = msToMidnight();

let timer = ref(convertMsToTime());

let counter = ref(0);

console.log("restOfDay", restOfDay);
let interval = null;

function amIOnStreak() {

  let userTime = Date.now() - user.value.timer;
  let streakTimeWindow = user.value.msToMidnight + 86400000;
  console.log("userTime:", userTime);
  console.log("streakWindow:", streakTimeWindow);

  if (userTime > streakTimeWindow) {
    counter.value = 0
    console.log("Counter:", counter.value)
  } else if (counter.value === 1) {
    console.log("One more and you're on a streak! Counter:", counter.value);
  } else {
    console.log("you're on a streak!")
    user.value.streak++
  }
}

function msToMidnight() {

  let offset2Hours = 2 * 60 * 60 * 1000;
  let oneDay = 86400000
  let msIntoDay = (Date.now() + offset2Hours) % oneDay;

  console.log("msintoDAy:", msIntoDay);
  console.log("oneday:", oneDay);
  console.log("tid igjen til midnatt:", oneDay - msIntoDay);

  return oneDay - msIntoDay;
}

function convertMsToTime() {

  const totalSeconds = Math.floor(msToMidnight() / 1000);
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

  startTimer()

})


async function updateScore() {
  console.log("update score")

  if (!user.value) {
    return;
  }
  console.log("milestone 1", milestones.value[0])
  let newMilestone;
  if (user.value.streak === 9 && !user.value.milestones.some(m => m.id === 1)) {
    newMilestone = milestones.value[0];
  } else if (user.value.streak === 19 && !user.value.milestones.some(m => m.id === 2)) {
    newMilestone = milestones.value[1];
  } else if (user.value.streak === 29 && !user.value.milestones.some(m => m.id === 3)) {
    newMilestone = milestones.value[2];
  }

  amIOnStreak();
  let newScore = (user.value.score ?? 0) + 1
  let newTime = Date.now();
  let toMidnight = msToMidnight();

  let req = await fetch("http://localhost:4002/user", {
    method: "PUT",
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
      ...user.value,
      score: newScore,
      streak: user.value.streak,
      timer: newTime,
      msToMidnight: toMidnight,
      counter: counter.value ?? 0,
      milestones: !newMilestone ? [...user.value.milestones] : [...user.value.milestones, newMilestone]
    })
  })
  user.value = await req.json();
}


function eachTick() {
// console.log("eachTick")
  timer.value = convertMsToTime();
}

function startTimer() {
  interval = setInterval(eachTick, 1000)
}

function stopTimer() {
  clearInterval(interval)
  interval = null;
}

</script>

<template>

  <div class="container">
    <h3 v-if="user">Hello {{ user.name }}<i class="fa-solid fa-user"/></h3>
    <div class="user-stats" v-if="user">
      <img v-if="user" :src="tjell" alt="profilePic" class="profilePic">
      <i class="fa-solid fa-star" style="color: #FFD43B;"></i> {{ user.score ?? 0 }} <i class="fa-solid fa-fire"
                                                                                        style="color: #FFD43B;"></i>
      {{ user.streak }}
    </div>
    <div v-if="user">
      <Modal :isOpen="isOpen">
        <div class="badges" v-for="m in user.milestones" :key="m.id">
          <div v-if="isOpen">
            <img :src="m.img" alt="img">
            <div class="text">{{ m.title }}</div>
          </div>
        </div>
        <button @click="isOpen = !isOpen">Close</button>
      </Modal>
    </div>

    <button :disabled="isAlreadyClickedToday" @click="updateScore">
      <span>
          {{ isAlreadyClickedToday ? "You have already clicked today" : "Check" }} <i class="fa-solid fa-thumbs-up"
                                                                                      style="margin-left: 5px; font-size: 30px"></i>
      </span>
    </button>
    <p><i class="fa-solid fa-stopwatch" style="padding: 5px; font-size: 30px"></i>Next Click Available In:</p>
    <strong>{{ timer.hours }} {{ timer.hours === 1 ? "Hour" : "Hours" }} - {{ timer.minutes }}
      {{ timer.minutes === 1 ? "Minute" : "Minutes" }} - {{ timer.seconds }}
      {{ timer.seconds === 1 ? "Second" : "Seconds" }}</strong>
    <button @click="stopTimer">Stop timer</button>
    <button @click="isOpen = !isOpen">
      <span>{{ isOpen ? "Close" : "Open Badges" }}</span>
    </button>
  </div>

</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  background: rgba(241, 232, 232, 0.2);
  padding: 30px;
  border-radius: 10px;
  box-shadow: gold 3px 2px 1px;
  text-align: center;
}

.user-stats {
  display: flex;
  justify-content: center;
  gap: 20px;
  font-size: 20px;
  padding: 30px;
}

button {
  box-shadow: #242424 3px 2px 1px;
}

strong {
  padding: 10px;
  margin-bottom: 10px;
}

.badges img {
  display: flex;
  flex-direction: row;
  height: 150px;
  width: 150px;
  margin-bottom: 10px;
}

.badges div {
  display: flex;
  justify-content: left;
  flex-direction: row;
}

.text {
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-align: center;
  font-weight: bold;
  color: gold;
}

.profilePic {
  border-radius: 100px;
  border-bottom: #242424 3px solid;
  height: 100px;
}


</style>
