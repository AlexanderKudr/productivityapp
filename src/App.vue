<script setup>
import Todo from './components/todos/Todo.vue';
import DarkMode from './components/darkMode/darkMode.vue';

// import Todo2 from './components/Todo2.vue';
import { ref, computed } from 'vue';

import { useProgressBar } from './hooks/useProgressBar'
import { useDarkMode } from '../src/hooks/useDarkMode';

import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { library } from '@fortawesome/fontawesome-svg-core'
import {
  faPlay,
  faPause,
  faArrowRotateRight,
  faChevronDown,
  faMoon
} from '@fortawesome/free-solid-svg-icons'
library.add(
  faPlay,
  faPause,
  faArrowRotateRight,
  faChevronDown,
  faMoon)
//fontawesome imports |

const { progressBarStart,
  progressBarPause,
  progressBarReset,
  progressBarDuration,
  progressBarUpdate,
  container,
} = useProgressBar();

const { dark, light, darkIsOn } = useDarkMode();

let timerRunning;
let btnToggle = ref(true);
// let whiteTheme = ref(true); white/dark theme setup later

const seconds = ref(0);
const display = ref("00 : 00")
let resetMinsTodo = ref(0);


const playButtonIcon = computed(() => {
  return btnToggle.value ? 'fa-play fa-solid' : 'fa-pause fa-solid'
})

function formatCode(secs) {
  const currentMinutes = Math.floor(secs / 60);
  const currentSeconds = secs % 60;
  return `${currentMinutes < 10 ? "0" + currentMinutes : currentMinutes}
          : ${currentSeconds < 10 ? "0" + currentSeconds : currentSeconds}`;
}

function countdownStart() {
  progressBarStart();
  progressBarUpdate();
  if (btnToggle.value === true) {
    btnToggle.value = false;
    clearInterval(timerRunning); // prevent timer from looping with a delay
    timerRunning = setInterval(() => {
      if (seconds.value > 0) {
        seconds.value--;
        display.value = formatCode(seconds.value);
      } else {
        countdownReset();
      }
    }, 1000)
  } else if (btnToggle.value === false) {
    clearInterval(timerRunning);
    btnToggle.value = true;
    progressBarPause();
  }
}

function countdownReset() {
  clearInterval(timerRunning);
  btnToggle.value = true;
  seconds.value = resetMinsTodo.value;
  display.value = formatCode(seconds.value);
  progressBarReset();
}

function todoTimeEvent(secs) {
  resetMinsTodo.value = secs;
  seconds.value = secs;
  display.value = formatCode(seconds.value);
  progressBarDuration(secs * 1000);
}

// let i = ref(0);
// function increment() {
//   i.value++;
// }

</script>

<template>

  <body class="background" :class="[darkIsOn ? dark : light]" >
    <header class="nav">
      <span class="header" :class="[darkIsOn ? dark : light]" >Productivity App</span>
    </header>
    <div class="app">
      <main class="main-app">
        <DarkMode></DarkMode>
        <div class="countdown">
          <div class="round-border" :class="[darkIsOn ? dark : light]">
            <div class="timer-background" :class="[darkIsOn ? dark : light]">
              <div ref="container" id="container"></div> <!-- ref is like creating an id (getelemenbyID) -->
              <span class="timer" :class="[darkIsOn ? dark : light]">{{ display }}</span>
              <div class="icons">
                <button class="icon1" @click="countdownStart()">
                  <font-awesome-icon :icon="playButtonIcon" size='2x' />
                </button>
                <button class="icon2" @click="countdownReset()">
                  <font-awesome-icon icon="fa-solid fa-arrow-rotate-right" size='2x' rotation="270" />
                </button>
              </div>
            </div>
          </div>
        </div>
      </main>
      <Todo 
      @reset-time-on-remove+check="countdownReset" 
      @test-event="todoTimeEvent">
      </Todo>
      <!-- <Todo @counter-update="increment"  
      :counter="i" /> -->
    </div>
  </body>
</template>

<style scoped lang="scss">
@use "../src/App.scss";
</style>

