<script setup>
import { ref, onMounted, onBeforeMount } from "vue";
import endOfSession from "@/assets/alarm.mp3";
let timer = null;
let audio = new Audio(endOfSession);
let isSettings = ref(false);
const modes = {
  pomodoro: ref({
    name: "pomodoro",
    time: 25,
    timer: 25 * 60,
    isRunning: false,
  }),
  short: ref({ name: "shortBreak", time: 5, timer: 5 * 60, isRunning: false }),
  long: ref({ name: "longBreak", time: 15, timer: 15 * 60, isRunning: false }),
};
let currentMode = ref("pomodoro");
let startPomodoro = () => {
  let mode = modes[currentMode.value].value;

  if (!mode.isRunning) {
    mode.isRunning = true;
    timer = setInterval(() => {
      if (mode.timer > 0) {
        mode.timer--;
      } else {
        clearInterval(timer);
        timer = null;
        mode.isRunning = false;
        audio.play();
      }
    }, 1000);
  } else {
    clearInterval(timer);
    timer = null;
    mode.isRunning = false;
  }
};
let resetPomodoro = () => {
  let mode = modes[currentMode.value].value;
  if (timer) {
    clearInterval(timer);
    timer = null;
  }
  mode.timer = mode.time * 60;
  mode.isRunning = false;
  audio.pause();
};
let saveSettings = ()=>{
  localStorage.setItem("savedSettings",JSON.stringify({
    pomodoro:modes["pomodoro"].value.time,
    short:modes["short"].value.time,
    long:modes["long"].value.time,
    currentMode:currentMode.value,
  }))
}
let goToSettings = () => {
  isSettings.value = !isSettings.value;
  modes["pomodoro"].value.timer = modes["pomodoro"].value.time * 60;
  modes["short"].value.timer = modes["short"].value.time * 60;
  modes["long"].value.timer = modes["long"].value.time * 60;
  saveSettings();
};
let switchModes = (chosenMode) => {
  currentMode.value = chosenMode;
  resetPomodoro();
  saveSettings();
};


onBeforeMount(() => {
  if (timer) clearInterval(timer);
});
onMounted(()=>{
  let saved = localStorage.getItem("savedSettings");
  if(saved){
    let parsed = JSON.parse(saved);
    modes["pomodoro"].value.time=parsed.pomodoro;
    modes["short"].value.time=parsed.short;
    modes["long"].value.time=parsed.long;
    modes["pomodoro"].value.timer=parsed.pomodoro*60;
    modes["short"].value.timer=parsed.short*60;
    modes["long"].value.timer=parsed.long*60;

    
  }
  if(currentMode){
    currentMode.value=parsed.currentMode;
  }
  
})

</script>
<template>
  <div class="navbar">
    <a href="/"
      ><h2><span id="navlogo">Pro</span>modoro</h2></a
    ><button @click="goToSettings" ><span class="material-symbols-outlined">settings</span></button>
  </div>
  <div class="settings" v-if="isSettings">
    <div class="settings-input">
      <h3>Pomodoro</h3>
      <input
        type="number"
        placeholder="25"
        min="0"
        step="1"
        v-model.number="modes['pomodoro'].value.time"
      />
    </div>
    <div class="settings-input">
      <h3>Short Break</h3>
      <input
        type="number"
        placeholder="5"
        min="0"
        step="1"
        v-model.number="modes['short'].value.time"
      />
    </div>
    <div class="settings-input">
      <h3>Long Break</h3>
      <input
        type="number"
        placeholder="15"
        min="0"
        step="1"
        v-model.number="modes['long'].value.time"
      />
    </div>
    <button @click="goToSettings" @keyup.enter="goToSettings"
    >Close</button>
  </div>

  <div class="timer-container">
    <div class="pages">
      <button  @click="switchModes('pomodoro')" :class="{changeMode:currentMode==='pomodoro'}">Pomodoro</button
      ><button @click="switchModes('short')" :class="{changeMode:currentMode==='short'}">Short Break</button
      ><button style="border: none" @click="switchModes('long')" :class="{changeMode:currentMode==='long'}">
        Long Break
      </button>
    </div>
    <h1>
      {{ Math.floor(modes[currentMode].value.timer / 60) }}:{{
        (modes[currentMode].value.timer % 60).toString().padStart(2, "0")
      }}
    </h1>
    <div class="timer-buttons">
      <button @click="startPomodoro" v-if="modes[currentMode].value.isRunning"><span class="material-icons-outlined">pause</span></button>
      <button @click="startPomodoro" v-else><span class="material-icons-outlined">play_arrow</span></button>
      <button @click="resetPomodoro"><span class="material-icons-outlined">cached</span></button>
       </div>
  </div>
</template>

