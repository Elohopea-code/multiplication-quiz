<template>
  <div>
    
    <h2>Valitse kertotaulut, joita haluat kysyä:</h2>
    <div v-if="isGameOver === false" class="quiz-buttons">        
      <!-- Predefined selection of tables (0-5 and 10) -->
      <button @click="selectPikku">Pikku</button>
      <!-- Predefined selection of all tables (0-10) -->
      <button @click="selectAll">Kaikki</button>
    </div>
    <!-- Table Selection: Choose which multiplication tables to quiz on -->
    <div v-if="!question">
      <div v-for="(table, index) in availableTables" :key="index">
        <input 
          type="checkbox" 
          :id="'table-' + table"
          :value="table"
          v-model="selectedTables" 
        />
        <label :for="'table-' + table" class="checkbox-label">{{ table }} kertotaulu</label>
      </div>

      <div>
        <!-- Start Quiz Button: Enabled only when at least one table is selected -->
        <button @click="startQuiz" :disabled="selectedTables.length === 0">Aloita kysely</button>
      </div>
    </div>

    <!-- Question Display: Shown when a question is active -->
    <div v-if="question">
      <div>
        <!-- Progress Bar: Displays the remaining time -->
        <progress :value="timeRemaining" :max="timeLimit"></progress>
        <!--<p>{{ timeRemaining / 1000 }} seconds remaining</p> -->
      </div>
      <h2>{{ question.num1 }} x {{ question.num2 }}</h2>
      <div>
        <!-- Answer Buttons: Allows the user to select an answer -->
        <button 
          v-for="(option, index) in answerOptions" 
          :key="index" 
          @click="checkAnswer(option)"
          :class="{
            'correct': isGameOver && option === correctAnswer, 
            'wrong': isGameOver && selectedAnswer === option && selectedAnswer !== correctAnswer 
          }"
          :disabled="isGameOver"
        >
          {{ option }}
        </button>
      </div>
      <div v-if="feedback !== null">
        <!-- Feedback: Displays whether the answer was correct or incorrect -->
        <p>{{ feedback }}</p>
        <p>Oikeat vastaukset peräkkäin: {{ streak }}</p>
        <p>Paras putki: {{ highScore }}</p>
      </div>
      <div v-if="isGameOver === true">
        <!-- Reset Quiz Button: Appears when the game is over -->
        <button @click="resetQuiz" v-if="isGameOver">Aloita alusta</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Available multiplication tables (0-10)
      availableTables: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
      // Selected multiplication tables
      selectedTables: [],
      // Time limit for each question in milliseconds
      timeLimit: 10000,
      // Current question object (num1, num2)
      question: null,
      // Answer options (including one correct answer and multiple wrong answers)
      answerOptions: [],
      // Timer for counting down the time
      timer: null,
      // Remaining time for the current question
      timeRemaining: this.timeLimit,
      // Feedback message (correct or incorrect answer)
      feedback: null,
      // Streak count (consecutive correct answers)
      streak: 0,
      // The answer selected by the user
      selectedAnswer: null,
      // The correct answer for the current question
      correctAnswer: null,
      // Flag indicating whether the game is over
      isGameOver: false,
      // The highest streak (best score)
      highScore: 0,
    };
  },
  methods: {
    // Starts the quiz by initializing necessary variables and generating a new question
    startQuiz() {
      this.selectedAnswer = null;
      this.correctAnswer = null;
      this.isGameOver = false;
      this.feedback = null;
      this.streak = 0;
      this.timeRemaining = this.timeLimit;
      this.generateQuestion();
      this.answerOptions = this.generateAnswerOptions();
      this.startTimer();
    },

    // Selects a predefined set of tables (0-5 and 10) and starts the quiz
    selectPikku() {
      this.selectedTables = [0, 1, 2, 3, 4, 5, 10];
      this.startQuiz();
    },

    // Selects a predefined set of all tables (0-10) and starts the quiz
    selectAll() {
      this.selectedTables = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
      this.startQuiz();
    },

    // Generates a random multiplication question based on the selected tables
    generateQuestion() {
      const table = this.selectedTables[Math.floor(Math.random() * this.selectedTables.length)];
      const num1 = table;
      const num2 = Math.floor(Math.random() + 1);
      this.question = { num1, num2 };
      this.correctAnswer = num1 * num2;
    },

    // Generates a list of 10 answer options (1 correct and 9 random wrong ones)
    generateAnswerOptions() {
      if (!this.question) return [];

      const correctAnswer = this.correctAnswer;
      const options = new Set([correctAnswer]);

      while (options.size < 10) {
        const wrongAnswer = Math.floor(Math.random() * 100);
        if (wrongAnswer !== correctAnswer) options.add(wrongAnswer);
      }

      return Array.from(options).sort(() => Math.random() - 0.5);
    },

    // Starts a countdown timer for the current question
    startTimer() {
      this.timer = setInterval(() => {
        if (this.timeRemaining > 0 && !this.isGameOver) {
          this.timeRemaining -= 1000;
        } else {
          clearInterval(this.timer);
          if (!this.isGameOver) {
            this.feedback = 'Aika loppui!';
            this.isGameOver = true;
            this.showCorrectAnswer(); // Show the correct answer if time runs out
          }
        }
      }, 1000);
    },

    // Checks the selected answer and updates feedback, streak, and game state
    checkAnswer(selectedAnswer) {
      this.selectedAnswer = selectedAnswer;
      const correctAnswer = this.correctAnswer;

      if (selectedAnswer === correctAnswer) {
        this.streak++;
        this.feedback = 'Oikein!';
        if (this.streak > this.highScore) {
          this.highScore = this.streak; // Update high streak score
        }
        setTimeout(() => {
          if (!this.isGameOver) {
            this.generateQuestion();
            this.answerOptions = this.generateAnswerOptions();
            this.timeRemaining = this.timeLimit;
          }
        }, 1000);
      } else {
        this.feedback = 'Väärin!';
        this.isGameOver = true;
        clearInterval(this.timer); // Stop the timer if the answer is incorrect
        this.showCorrectAnswer(); // Show the correct answer
      }
    },

    // Highlights the correct and selected answers after the game is over
    showCorrectAnswer() {
      const buttons = document.querySelectorAll('button');
      buttons.forEach(button => {
        if (button.textContent == this.correctAnswer) {
          button.classList.add('correct');
        } else if (button.textContent == this.selectedAnswer) {
          button.classList.add('wrong');
        }
      });
    },

    // Resets the quiz, clearing all data and reinitializing the game
    resetQuiz() {
      clearInterval(this.timer);
      this.selectedTables = [];
      this.feedback = null;
      this.question = null;
      this.timeRemaining = this.timeLimit;
      this.streak = 0;
      this.isGameOver = false;
      this.correctAnswer = null;
    },
  },
};
</script>

<style>
/* Correct answer button style */
button.correct {
  background-color: #4caf50; /* Green color for correct answer */
  color: white;
}

/* Incorrect answer button style */
button.wrong {
  background-color: #f44336; /* Red color for incorrect answer */
  color: white;
}

/* Disabled button style (un-clickable) */
button:disabled {
  cursor: not-allowed;
}

/* Focus and active button styles */
button:focus, button:active {
  outline: none;
  box-shadow: none;
}

/* Progress bar styling */
progress {
  width: 100%;
  height: 20px;
}

/* General button padding */
select {
  padding: 5px;
}

/*  Larger text next to the checkboxes */
.checkbox-label {
  font-size: 1.25rem;
  cursor: pointer;
  display: inline-block; 
  line-height: 1rem; 
  padding: 0 0.5rem; 
}

/* Button styling */
button {
  margin: 1rem;
}

/* More space after the quiz buttons */
.quiz-buttons {
  margin-bottom: 1rem;
}

</style>
