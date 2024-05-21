<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

interface FlashcardSets {
  [key: string]: string[]
}

const flashcardSets = ref<FlashcardSets>({})
const selectedSet = ref('')
const deck = ref<string[]>([])
const finished = ref<string[]>([])

const currentCard = computed(() => {
  if (deck.value.length === 0) {
    return 'Done!'
  } else {
    const card = deck.value[0]
    return selectedSet.value === 'alphabet' ? `${card} ${card.toLowerCase()}` : card
  }
})

async function fetchFlashcardSets(): Promise<void> {
  try {
    const response = await axios.get<FlashcardSets>('flashcards.json')
    flashcardSets.value = response.data
    selectedSet.value = Object.keys(flashcardSets.value)[0]
    resetCards()
  } catch (error) {
    console.error('Error fetching flashcard sets:', error)
  }
}

function resetCards(): void {
  const set = selectedSet.value
  console.log(flashcardSets.value[set])
  deck.value = [...flashcardSets.value[set]].sort(() => Math.random() - 0.5)
  finished.value = []
}

function handleKeydown(event: KeyboardEvent): void {
  switch (event.key) {
    case 'ArrowRight':
      if (deck.value.length > 0) {
        finished.value.push(deck.value.shift()!)
      }
      break
    case 'ArrowLeft':
      if (deck.value.length > 0) {
        deck.value.splice(5, 0, deck.value.shift()!)
      }
      break
    case 'ArrowUp':
    case 'Enter': {
      const speech = new SpeechSynthesisUtterance(deck.value[0].toLocaleLowerCase())
      speech.lang = 'en-US'
      speech.rate = 0.8
      speech.pitch = 1
      window.speechSynthesis.speak(speech)
      break
    }
    case 'Escape':
      resetCards()
      break
  }
}

onMounted(() => {
  fetchFlashcardSets()
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <main>
    <div id="app">
      <div class="header">
        <h1>Flashcard App</h1>
        <select v-model="selectedSet" @change="resetCards" class="set-selector">
          <option v-for="(set, index) in flashcardSets" :key="index" :value="index">
            {{ index }}
          </option>
        </select>
      </div>

      <div class="content">
        <div id="flashCard">{{ currentCard }}</div>
        <div class="metadata">
          <div id="leftCount">Left: {{ deck.length }}</div>
          <div id="doneCount">Done: {{ finished.length }}</div>
        </div>
      </div>

      <div class="instructions">
        <h2>Instructions</h2>
        <ul>
          <li><strong>→</strong> Got it</li>
          <li><strong>←</strong> Missed it</li>
          <li><strong>Enter</strong> Hint</li>
          <li><strong>Esc</strong> Reset</li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
#app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

h1 {
  font-size: 24px;
  margin: 0;
}

.set-selector {
  padding: 8px 12px;
  font-size: 16px;
}

.content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

#flashCard {
  min-width: 400px;
  min-height: 300px;
  font-size: 48px;
  text-align: center;
  background-color: var(--color-background-soft);
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 20px;
}

.metadata {
  display: flex;
  justify-content: space-between;
  width: 100%;
  font-size: 18px;
  color: #888;
}

#leftCount,
#doneCount {
  margin: 0;
}

.instructions {
  margin-top: 40px;
}

h2 {
  font-size: 20px;
  margin-bottom: 10px;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

li {
  margin-bottom: 5px;
}

li strong {
  display: inline-block;
  width: 60px;
  font-weight: bold;
}
</style>
