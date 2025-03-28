<template>
  <div class="quiz-container">
    <h1> {{ name }}</h1>
    <div v-if="!quizFinished">
      <div v-for="(question, index) in questions" :key="index" class="question">
        <QuizQuestion
          :question="question"
          :questionIndex="index"
          :selectedAnswer="userAnswers[index]"
          @answer-selected="updateAnswer(index, $event)"
        />
      </div>
      <button @click="submitQuiz">Submit Challenge</button>
    </div>

    <div v-else>
      <h3>Results</h3>
      <div v-for="(question, index) in questions" :key="index">
        <p><strong>Question:</strong> {{ question.title }}</p>
        <!-- Show all options for the question with highlighting the correct answer -->
        <div v-for="(option, optionIndex) in question.options" :key="optionIndex">
          <p :style="getOptionStyle(option, question.correctAnswer, userAnswers[index])">
            <strong>{{ option }}</strong>
          </p>
        </div>
        <p><strong>Correct Answer:</strong> {{ question.correctAnswer }}</p>
        <p v-if="!isCorrect(index)" ref="explanation">
          <strong>Explanation:</strong> {{ question.explanation }}
        </p>
        <p v-if="isCorrect(index)" style="color: #c8e6ce;">
          Correct! Well done!
        </p>
        <p v-else style="color: red;">
          Incorrect. Better luck next time!
        </p>
      </div>
      <div>
        <h4>Your Score: {{ score }} / {{ questions.length }}</h4>
        <span>You earned {{ score }} credits!</span>
      </div>
      <!--<button @click="restartQuiz">Try Again</button>-->
    </div>
  </div>
</template>

<script>
import QuizQuestion from "~/components/challenge/QuizQuestion.vue";
import renderMathInElement from "katex/contrib/auto-render/auto-render.js";

export default {
  components: {
    QuizQuestion
  },
  props: {
    quiz: {
      type: Object
    },
  },
  data() {
    return {
      questions: this.quiz,
      questions2: [{
        "id": 1,
        "title": "Which of the following is a characteristic of living organisms?",
        "options": ["Reproduction", "Nutrition", "Excretion", "All of the above"],
        "correctAnswer": "All of the above"
      }, {
        "id": 2,
        "title": "What is the function of the cell membrane?",
        "options": ["To regulate the movement of substances into and out of the cell", "To synthesize proteins", "To store genetic information", "To carry out respiration"],
        "correctAnswer": "To regulate the movement of substances into and out of the cell"
      }],
      userAnswers: [],
      quizFinished: false,
      score: 0,
      name: this.$route.query.name
    };
  },
  methods: {
    submitQuiz() {
      this.quizFinished = true;
      this.calculateScore();
      this.$emit('quiz-submitted', this.score);
    },
    updateAnswer(index, answer) {
      this.userAnswers[index] = answer;
    },
    isCorrect(index) {
      return this.userAnswers[index] + "" === this.questions[index].correctAnswer;
    },
    calculateScore() {
      this.score = this.userAnswers.reduce((total, answer, index) => {
        return total + (answer === this.questions[index].correctAnswer ? 1 : 0);
      }, 0);
    },
    restartQuiz() {
      this.userAnswers = [];
      this.score = 0;
      this.quizFinished = false;
    },
    getOptionStyle(option, correctAnswer, userAnswer) {
      // Set the option style based on whether it's correct, incorrect or the user's selected answer
      if (option === correctAnswer) {
        return { color: 'green', fontWeight: 'bold' }; // Correct option in green
      }
      if (option === userAnswer) {
        return { color: 'red', fontWeight: 'bold' }; // User's incorrect answer in red
      }
      return {}; // No special style for other options
    },
    renderMath(element) {
      try {
        return renderMathInElement(element, {
          delimiters: [
            {left: "$$", right: "$$", display: false},
            {left: "\\(", right: "\\)", display: false},
            // LaTeX uses $…$, but it ruins the display of normal `$` in text:
            {left: "\\newline", right: "\\newline", display: true},
            // $ must come after $$
            // Render AMS environments even if outside $$…$$ delimiters.
            {left: "\\begin{equation}", right: "\\end{equation}", display: false},
            {left: "\\begin{align}", right: "\\end{align}", display: false},
            {left: "\\begin{alignat}", right: "\\end{alignat}", display: false},
            {left: "\\begin{gather}", right: "\\end{gather}", display: false},
            {left: "\\begin{CD}", right: "\\end{CD}", display: false},
            {left: "\\[", right: "\\]", display: true},
          ]
        });
      } catch (error) {
        console.error('KaTeX rendering error:', error);
        return element;
      }
    }
  },
  mounted() {
    if (this.question && this.question.explanation) {
      this.renderMath(this.$refs.explanation);
    }
  }
};
</script>

<style scoped>
.quiz-container {
  width: 70%;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

h3 {
  font-size: 1.2em;
}

h4 {
  margin-top: 20px;
  font-size: 1.2em;
  font-weight: bold;
}

.explanation {
  color: #48bb78;
  font-weight: bold;
  display: block;
  margin-top: 10px;
  cursor: pointer;
}

.explanation:hover {
  text-decoration: underline;
}

.question {
  margin-top: 10px;
}

hr {
  margin: 20px 0;
}

p {
  margin: 5px 0;
}

strong {
  font-weight: bold;
}
</style>
