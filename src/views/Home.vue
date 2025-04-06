<template>
  <div class="home">
    <div class="practice-container">
      <div class="word-display">
        <h2>Listen and Type the Word</h2>
        <button @click="playWord" class="play-button" :disabled="isPlaying">
          <span class="play-icon">🔊</span> Play Word
        </button>
      </div>

      <div class="input-section">
        <input 
          v-model="userInput" 
          @keyup.enter="checkAnswer"
          placeholder="Type the word here..."
          class="word-input"
          ref="wordInput"
        >
        <button @click="checkAnswer" class="submit-button" :disabled="!userInput.trim()">Submit</button>
      </div>

      <div class="result-section" v-if="showResult">
        <p :class="['result-message', isCorrect ? 'correct' : 'incorrect']">
          {{ isCorrect ? 'Correct!' : 'Incorrect!' }}
        </p>
        <p v-if="!isCorrect" class="correct-answer">
          The correct word was: <strong>{{ currentWord }}</strong>
        </p>
      </div>

      <div class="controls">
        <button @click="nextWord" class="next-button">Next Word</button>
      </div>
    </div>

    <div class="history-container">
      <h3>Recent Words</h3>
      <div class="history-list">
        <div v-for="(word, index) in recentWords" :key="index" class="history-item">
          <span class="word-text">{{ word.word }}</span>
          <span :class="['result-indicator', word.correct ? 'correct' : 'incorrect']">
            {{ word.correct ? '✓' : '✗' }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      words: [
        { word: 'apple' },
        { word: 'banana' },
        { word: 'orange' },
        { word: 'grape' },
        { word: 'mango' },
        { word: 'computer' },
        { word: 'phone' },
        { word: 'book' },
        { word: 'pencil' },
        { word: 'desk' },
        { word: 'laptop' },
        { word: 'chair' },
        { word: 'table' },
        { word: 'window' },
        { word: 'door' }
      ],
      currentWordIndex: 0,
      userInput: '',
      showResult: false,
      isCorrect: false,
      isPlaying: false,
      speechSynthesis: null,
      recentWords: [],
      availableIndices: []
    }
  },
  computed: {
    currentWord() {
      return this.words[this.currentWordIndex].word
    }
  },
  methods: {
    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    },
    initializeAvailableIndices() {
      this.availableIndices = Array.from({ length: this.words.length }, (_, i) => i);
      this.shuffleArray(this.availableIndices);
    },
    getNextWordIndex() {
      if (this.availableIndices.length === 0) {
        this.initializeAvailableIndices();
      }
      return this.availableIndices.pop();
    },
    playWord() {
      if (this.isPlaying) return
      
      this.isPlaying = true
      this.showResult = false
      
      // Cancel any ongoing speech
      if (this.speechSynthesis) {
        window.speechSynthesis.cancel()
      }

      // Create new speech utterance
      const utterance = new SpeechSynthesisUtterance(this.currentWord)
      utterance.lang = 'en-US'
      utterance.rate = 0.8
      utterance.pitch = 1
      utterance.volume = 1

      // Handle speech end
      utterance.onend = () => {
        this.isPlaying = false
      }

      // Speak the word
      window.speechSynthesis.speak(utterance)
      this.speechSynthesis = utterance
    },
    checkAnswer() {
      if (!this.userInput.trim()) return
      
      this.showResult = true
      this.isCorrect = this.userInput.toLowerCase().trim() === this.currentWord
      
      // Add to recent words history
      this.recentWords.unshift({
        word: this.currentWord,
        correct: this.isCorrect
      })
      
      // Keep only the last 5 words
      if (this.recentWords.length > 5) {
        this.recentWords.pop()
      }
    },
    nextWord() {
      this.currentWordIndex = this.getNextWordIndex()
      this.showResult = false
      this.userInput = ''
      this.isPlaying = false
      
      // Cancel any ongoing speech
      if (this.speechSynthesis) {
        window.speechSynthesis.cancel()
      }
      
      // Focus the input field
      this.$nextTick(() => {
        this.$refs.wordInput.focus()
      })
    }
  },
  mounted() {
    // Initialize available indices when component is mounted
    this.initializeAvailableIndices()
    // Focus the input field when component is mounted
    this.$nextTick(() => {
      this.$refs.wordInput.focus()
    })
  },
  beforeDestroy() {
    // Clean up speech synthesis when component is destroyed
    if (this.speechSynthesis) {
      window.speechSynthesis.cancel()
    }
  }
}
</script>

<style scoped>
.home {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  display: flex;
  gap: 2rem;
}

.practice-container {
  flex: 2;
  background-color: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.history-container {
  flex: 1;
  background-color: white;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  height: fit-content;
}

.history-container h3 {
  margin-top: 0;
  color: #2c3e50;
  font-size: 1.2rem;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.history-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  background-color: #f8f9fa;
  border-radius: 4px;
}

.word-text {
  font-weight: 500;
}

.result-indicator {
  font-size: 1.2rem;
}

.result-indicator.correct {
  color: #42b983;
}

.result-indicator.incorrect {
  color: #e74c3c;
}

.word-display {
  margin-bottom: 2rem;
}

.play-button {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1.1rem;
  transition: background-color 0.3s;
}

.play-button:disabled {
  background-color: #a8d5c2;
  cursor: not-allowed;
}

.play-button:not(:disabled):hover {
  background-color: #3aa876;
}

.input-section {
  margin-bottom: 1.5rem;
}

.word-input {
  width: 100%;
  padding: 10px;
  font-size: 1.1rem;
  border: 2px solid #ddd;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.word-input:focus {
  outline: none;
  border-color: #42b983;
}

.submit-button {
  background-color: #4a90e2;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.submit-button:disabled {
  background-color: #a8c7e2;
  cursor: not-allowed;
}

.submit-button:not(:disabled):hover {
  background-color: #357abd;
}

.result-section {
  margin: 1.5rem 0;
}

.result-message {
  font-size: 1.2rem;
  font-weight: bold;
}

.correct {
  color: #42b983;
}

.incorrect {
  color: #e74c3c;
}

.correct-answer {
  color: #666;
  margin-top: 0.5rem;
}

.controls {
  margin-top: 2rem;
}

.next-button {
  background-color: #666;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.next-button:hover {
  background-color: #555;
}
</style> 