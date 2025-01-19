<template>
  <main class="flex justify-center items-center h-screen flex-col font-mono font-bold">
    <div class="rounded border-2 border-gray-300 p-4 w-6/12 h-2/4 flex flex-col">
      <section class="flex justify-center items-center mb-4">
        <h1 class="text-3xl">SwiftType</h1>
        <span></span>
      </section>
      <section class="flex-grow border border-gray-300 p-4 rounded overflow-y-auto">
        <div class="flex flex-wrap gap-2">
          <!--Words and Characters Will Be Displayed Here-->
        </div>
      </section>
      <section class="flex justify-between text-xl mb-2 mt-2">
        <p class="flex-1 text-left">Time: {{ state.Time }}s</p>
        <p class="flex-1 text-center">WPM: {{ state.WPM }}</p>
        <p class="flex-1 text-right">Accuracy: {{ state.Accuracy }}%</p>
      </section>
      <section>
        <button class="btn btn-outline text-2xl w-full rounded" @click="toggleGame">
          {{ state.GameInSession ? 'Restart' : 'Start' }}
        </button>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import { reactive } from 'vue'

const state = reactive({
  GameInSession: false,
  words: [],
  Time: 0,
  WPM: 0,
  Accuracy: 0,
})

function toggleGame() {
  state.GameInSession = !state.GameInSession
  if (state.GameInSession) {
    startGame()
  } else {
    restartGame()
  }
}

function startGame() {
  state.GameInSession = true
}

function restartGame() {
  state.GameInSession = false
  state.Time = 0
  state.WPM = 0
  state.Accuracy = 0
}
</script>
