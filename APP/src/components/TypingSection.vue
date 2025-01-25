<template>
  <main class="flex justify-center items-center h-full flex-col font-mono font-bold mt-[-68px]">
    <div class="rounded border-2 border-gray-300 p-4 w-6/12 h-2/4 flex flex-col">
      <!--Title Stuff-->
      <section class="flex justify-center items-center mb-4">
        <h1 class="text-3xl">SwiftType</h1>
        <span></span>
      </section>
      <!--New number of word selector! With custom inputs! I love myself.-->
      <section class="mb-4 flex justify-center items-center space-x-2">
        <select
          v-model="selectedWordCount"
          @change="handleWordCountChange"
          class="p-2 border rounded"
        >
          <option value="custom">Custom</option>
          <option v-for="count in predefinedWordCounts" :key="count" :value="count">
            {{ count }} words
          </option>
        </select>
        <input
          v-if="selectedWordCount === 'custom'"
          v-model="customWordCount"
          @input="handleCustomWordCountInput"
          type="number"
          min="1"
          max="1000"
          class="p-2 border rounded w-24"
          placeholder="Words"
        />
      </section>
      <!--Text Container stuff-->
      <section
        ref="textContainer"
        class="flex-grow border border-gray-300 p-4 rounded overflow-y-auto"
      >
        <div class="flex flex-wrap text-2xl">
          <div v-for="(word, wordIndex) in state.words" :key="wordIndex">
            <span
              v-for="(char, charIndex) in word"
              :key="charIndex"
              class="p-[1px] transition-all duration-300 w-6"
              :class="{
                'border-2 border-gray-300': isCurrentChar(wordIndex, charIndex),
                'border-2 border-transparent': !isCurrentChar(wordIndex, charIndex),
                'text-green-500': isCorrectChar(wordIndex, charIndex),
                'text-red-500': isIncorrectChar(wordIndex, charIndex),
              }"
            >
              {{ char }}
            </span>
            <span
              class="p-[1px] transition-all duration-300"
              :class="{
                'border-2 border-gray-300': isCurrentChar(wordIndex, word.length),
                'border-2 border-transparent':
                  !isCurrentChar(wordIndex, word.length) || state.IsCompleted,
                'text-green-500': isCorrectChar(wordIndex, word.length),
                'text-red-500': isIncorrectChar(wordIndex, word.length),
              }"
              > 
            </span>
          </div>
        </div>
      </section>
      <!--Stats Section-->
      <section class="flex justify-between text-xl mb-2 mt-2">
        <p class="flex-1 text-left">Time: {{ state.Time }}s</p>
        <p class="flex-1 text-center">WPM: {{ state.WPM }}</p>
        <p class="flex-1 text-right">Accuracy: {{ state.Accuracy }}%</p>
      </section>
      <!--Start/Restart Button. Y'know adding these comments might be a little inefficient. There could be something I can do... but I'm far too lazy....-->
      <section>
        <button
          class="btn btn-outline text-2xl w-full rounded bg-transparent border-2 border-white hover:scale-[1.015] focus:outline-none"
          @click="toggleGame"
          @keydown.prevent
        >
          {{ state.GameInSession ? 'Restart' : 'Start' }}
        </button>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
// I'm like 99% sure this can be simplified to be more efficient but I'm lowkey too dumb to actually do it... maybe at a future date... why fix what's not broke? Y'know what I mean? Actually, I'm not all too sure if I know what I mean...
import { reactive, ref, onMounted, watch } from 'vue'

interface State {
  GameInSession: boolean
  words: string[]
  Time: number
  WPM: number
  Accuracy: number
  Wrong: number
  Correct: number
  CurrentIndex: [number, number]
  Ticking: boolean
  CharStatus: { [key: string]: 'correct' | 'incorrect' | null }
  IsCompleted: boolean
  TotalAttempts: number
}

const state = reactive<State>({
  GameInSession: false,
  words: [],
  Time: 0,
  WPM: 0,
  Accuracy: 0,
  Wrong: 0,
  Correct: 0,
  CurrentIndex: [0, 0],
  Ticking: false,
  CharStatus: {},
  IsCompleted: false,
  TotalAttempts: 0,
})

const textContainer = ref<HTMLElement | null>(null)
const selectedWordCount = ref<number | 'custom'>(20)
const customWordCount = ref<number | null>(null)
const predefinedWordCounts = [10, 20, 30, 50, 100]

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
  retrieveWords()
})

watch(customWordCount, (newValue) => {
  if (newValue !== null && !state.GameInSession) {
    retrieveWords()
  }
})

function handleKeyDown(e: KeyboardEvent) {
  if (!/^[a-zA-Z ]$/.test(e.key) && e.key !== 'Backspace') return
  if (!state.GameInSession && e.key !== ' ') {
    toggleGame()
  }
  if (!state.IsCompleted) {
    handleInput(e)
  }
}

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
  state.CurrentIndex = [0, 0]
  state.CharStatus = {}
  state.IsCompleted = false
  state.Time = 0
  state.WPM = 0
  state.Accuracy = 0
  state.Wrong = 0
  state.Correct = 0
  state.Ticking = false
  state.TotalAttempts = 0
}

function restartGame() {
  state.GameInSession = false
  state.Ticking = false
  state.Time = 0
  state.WPM = 0
  state.Accuracy = 0
  state.CurrentIndex = [0, 0]
  state.Wrong = 0
  state.Correct = 0
  state.CharStatus = {}
  state.IsCompleted = false
  state.TotalAttempts = 0
  retrieveWords()
}

function calculateStats() {
  state.WPM = Math.round(state.Correct / 5 / (state.Time / 60))
  state.Accuracy = Math.round((state.Correct / state.TotalAttempts) * 100)
}

async function startTimeTicker() {
  // Using promise looks much cleaner than setInterval tbh. Also, it's easier to stop.... yeah... I think I'll stick with this.
  while (state.GameInSession && !state.IsCompleted) {
    await new Promise((resolve) => setTimeout(resolve, 10))
    if (state.GameInSession && !state.IsCompleted) {
      state.Time = Math.round((state.Time += 0.01) * 100) / 100
      calculateStats()
    }
  }
}

function handleInput(input: KeyboardEvent) {
  // There is probably something I CAN do to make this function cleaner, but I'm not sure what it is. I'll leave it as is for now. It's not too bad. I think. I hope. I'm not sure. I'm not confident in my code. I'm not confident in myself. I'm not confident in anything. I'm not confident in my ability to code. I'm not confident in my ability to do anything.
  if (!state.Ticking) {
    state.Ticking = true
    startTimeTicker()
  }

  const [wordIndex, charIndex] = state.CurrentIndex
  const currentWord = state.words[wordIndex]

  if (input.key === 'Backspace') {
    if (charIndex > 0) {
      state.CurrentIndex = [wordIndex, charIndex - 1]
      state.CharStatus[`${wordIndex}-${charIndex - 1}`] = null
    } else if (wordIndex > 0) {
      state.CurrentIndex = [wordIndex - 1, state.words[wordIndex - 1].length]
      state.CharStatus[`${wordIndex - 1}-${state.words[wordIndex - 1].length}`] = null
    }
    return
  }

  state.TotalAttempts++

  if (charIndex < currentWord.length) {
    if (input.key === currentWord[charIndex]) {
      state.Correct++
      state.CharStatus[`${wordIndex}-${charIndex}`] = 'correct'
    } else {
      state.Wrong++
      state.CharStatus[`${wordIndex}-${charIndex}`] = 'incorrect'
    }
    state.CurrentIndex = [wordIndex, charIndex + 1]
  } else if (charIndex === currentWord.length && input.key === ' ') {
    if (wordIndex < state.words.length - 1) {
      state.CurrentIndex = [wordIndex + 1, 0]
      state.Correct++
      state.CharStatus[`${wordIndex}-${charIndex}`] = 'correct'
      scrollToCurrentWord()
    } else {
      state.IsCompleted = true
      state.GameInSession = false
      state.CharStatus[`${wordIndex}-${charIndex}`] = 'correct'
    }
  } else {
    state.Wrong++
    state.CharStatus[`${wordIndex}-${charIndex}`] = 'incorrect'
  }
}

function scrollToCurrentWord() {
  //Lowkey bashed my head against the wall for like 30 minutes trying to figure out why this wasn't working. Turns out I forgot to add the value property to the ref. I'm a genius. Totally didn't get any help. I'm just that smart. I'm just that good at coding. I'm just that good at everything. I'm just that good at life.
  if (textContainer.value) {
    const [wordIndex] = state.CurrentIndex
    const words = textContainer.value.querySelectorAll('div > div')
    if (words[wordIndex]) {
      const containerRect = textContainer.value.getBoundingClientRect()
      const wordRect = words[wordIndex].getBoundingClientRect()
      const scrollAmount =
        wordRect.top - containerRect.top - containerRect.height / 2 + wordRect.height / 2

      textContainer.value.scrollBy({
        top: scrollAmount,
        behavior: 'smooth',
      })
    }
  }
}

// I'm not sure if this function is necessary, but I'm too scared to remove it. It works with both of them in it... so... I'll just leave it. Why fix something that's not broke? Y'know what I mean?
function isCurrentChar(wordIndex: number, charIndex: number): boolean {
  const [currentWordIndex, currentCharIndex] = state.CurrentIndex
  if (wordIndex === currentWordIndex) {
    if (charIndex === state.words[wordIndex].length) {
      return currentCharIndex === charIndex
    }
    return currentCharIndex === charIndex
  }
  return false
}

function isCorrectChar(wordIndex: number, charIndex: number): boolean {
  return state.CharStatus[`${wordIndex}-${charIndex}`] === 'correct'
}

function isIncorrectChar(wordIndex: number, charIndex: number): boolean {
  return state.CharStatus[`${wordIndex}-${charIndex}`] === 'incorrect'
}

async function retrieveWords() {
  const wordCount =
    selectedWordCount.value === 'custom' ? customWordCount.value : selectedWordCount.value
  if (wordCount === null || wordCount < 1) return

  const URL = `https://random-word-api.herokuapp.com/word?number=${wordCount}`
  const response = await fetch(URL)
  const data = await response.json()
  state.words = data.map((word: string) => word.split(''))
}

function handleWordCountChange() {
  if (selectedWordCount.value !== 'custom') {
    customWordCount.value = null
  }
  if (!state.GameInSession) {
    retrieveWords()
  }
}

function handleCustomWordCountInput() {
  if (customWordCount.value !== null) {
    customWordCount.value = Math.max(1, Math.min(1000, customWordCount.value))
  }
}
</script>
