<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

interface Session {
  id: string
  start: string
  end: string
  deck: string[]
  finished: string[]
  missed: string[]
}

interface FlashcardSets {
  [key: string]: string[]
}

const currentSession = ref<Session>({
  id: '',
  start: '',
  end: '',
  deck: [],
  finished: [],
  missed: []
})
const flashcardSets = ref<FlashcardSets>({})
const selectedSet = ref('')

const currentCard = computed(() => {
  if (currentSession.value.deck.length === 0) {
    return 'Done!'
  } else {
    return currentSession.value.deck[0]
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
  currentSession.value = {
    id: Date.now().toString(),
    start: new Date().toISOString(),
    end: '',
    deck: [...flashcardSets.value[selectedSet.value]].sort(() => Math.random() - 0.5),
    finished: [],
    missed: []
  }
}

function handleKeydown(event: KeyboardEvent): void {
  switch (event.key) {
    case 'ArrowRight':
      if (currentSession.value.deck.length > 0) {
        currentSession.value.finished.push(currentSession.value.deck.shift()!)
      }
      break
    case 'ArrowLeft':
      if (currentSession.value.deck.length > 0) {
        currentSession.value.missed.push(currentSession.value.deck[0])
        currentSession.value.deck.splice(5, 0, currentSession.value.deck.shift()!)
      }
      break
    case 'ArrowUp':
    case 'Enter': {
      const speech = new SpeechSynthesisUtterance(currentSession.value.deck[0].toLocaleLowerCase())
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
  <div class="home">
    <div class="header">
      <h1>Flashcard</h1>
      <select v-model="selectedSet" @change="resetCards" class="set-selector">
        <option v-for="(set, index) in flashcardSets" :key="index" :value="index">
          {{ index }}
        </option>
      </select>
    </div>

    <div class="content">
      <div id="flashCard">{{ currentCard }}</div>
      <div class="metadata">
        <div id="leftCount">Left: {{ currentSession.deck.length }}</div>
        <div id="doneCount">Done: {{ currentSession.finished.length }}</div>
      </div>
    </div>

    <div class="instructions">
      <ul>
        <li><strong>→</strong> Got it</li>
        <li><strong>←</strong> Missed it</li>
        <li><strong>Enter</strong> Hint</li>
        <li><strong>Esc</strong> Reset</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
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
  height: 300px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
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

#deckCount,
#doneCount {
  margin: 0;
}

.instructions {
  margin-top: 20px;
  ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
    font-size: 10px;
  }

  li {
    margin-bottom: 0;
  }

  li strong {
    display: inline-block;
    width: 40px;
    font-weight: bold;
  }
}
</style>
