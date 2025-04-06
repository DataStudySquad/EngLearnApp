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
        { word: 'desk' }
      ],
      currentWordIndex: 0,
      userInput: '',
      showResult: false,
      isCorrect: false,
      isPlaying: false,
      speechSynthesis: null
    }
  },
  computed: {
    currentWord() {
      return this.words[this.currentWordIndex].word
    }
  },
  methods: {
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
    },
    nextWord() {
      this.currentWordIndex = (this.currentWordIndex + 1) % this.words.length
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
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.practice-container {
  background-color: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
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