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
  max-width: 1000px;
  margin: 0 auto;
  padding: 2rem;
  display: flex;
  gap: 2.5rem;
  min-height: 100vh;
  background-color: #f5f7fa;
}

.practice-container {
  flex: 2;
  background-color: white;
  border-radius: 16px;
  padding: 2.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease;
}

.practice-container:hover {
  transform: translateY(-2px);
}

.history-container {
  flex: 1;
  background-color: white;
  border-radius: 16px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  height: fit-content;
  position: sticky;
  top: 2rem;
}

.history-container h3 {
  margin-top: 0;
  color: #2c3e50;
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #f0f0f0;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.history-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background-color: #f8f9fa;
  border-radius: 12px;
  transition: all 0.2s ease;
}

.history-item:hover {
  background-color: #f0f2f5;
  transform: translateX(4px);
}

.word-text {
  font-weight: 500;
  font-size: 1.2rem;
  color: #2c3e50;
}

.result-indicator {
  font-size: 1.4rem;
  font-weight: bold;
}

.result-indicator.correct {
  color: #42b983;
}

.result-indicator.incorrect {
  color: #e74c3c;
}

.word-display {
  margin-bottom: 2.5rem;
  text-align: center;
}

.word-display h2 {
  font-size: 2rem;
  color: #2c3e50;
  margin-bottom: 1.5rem;
  font-weight: 600;
}

.play-button {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.3rem;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  margin: 0 auto;
  min-width: 200px;
}

.play-button:disabled {
  background-color: #a8d5c2;
  cursor: not-allowed;
  transform: none;
}

.play-button:not(:disabled):hover {
  background-color: #3aa876;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(66, 185, 131, 0.2);
}

.input-section {
  margin-bottom: 2rem;
}

.word-input {
  width: 100%;
  padding: 1rem 1.2rem;
  font-size: 1.3rem;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  margin-bottom: 1rem;
  transition: all 0.3s ease;
}

.word-input:focus {
  outline: none;
  border-color: #42b983;
  box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.1);
}

.submit-button {
  background-color: #4a90e2;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.2rem;
  transition: all 0.3s ease;
  width: 100%;
}

.submit-button:disabled {
  background-color: #a8c7e2;
  cursor: not-allowed;
  transform: none;
}

.submit-button:not(:disabled):hover {
  background-color: #357abd;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.2);
}

.result-section {
  margin: 2rem 0;
  text-align: center;
  padding: 1.5rem;
  border-radius: 12px;
  background-color: #f8f9fa;
}

.result-message {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.correct {
  color: #42b983;
}

.incorrect {
  color: #e74c3c;
}

.correct-answer {
  color: #666;
  font-size: 1.2rem;
  margin-top: 0.8rem;
}

.controls {
  margin-top: 2.5rem;
  text-align: center;
}

.next-button {
  background-color: #666;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.2rem;
  transition: all 0.3s ease;
  min-width: 200px;
}

.next-button:hover {
  background-color: #555;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 102, 102, 0.2);
}

@media (max-width: 768px) {
  .home {
    flex-direction: column;
    padding: 1rem;
  }

  .history-container {
    position: static;
    margin-top: 2rem;
  }

  .practice-container,
  .history-container {
    padding: 1.5rem;
  }

  .word-display h2 {
    font-size: 1.8rem;
  }

  .play-button,
  .submit-button,
  .next-button {
    font-size: 1.1rem;
    padding: 0.8rem 1.5rem;
  }

  .word-input {
    font-size: 1.1rem;
    padding: 0.8rem 1rem;
  }
}
</style> 