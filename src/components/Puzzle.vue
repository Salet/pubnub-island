<template>
  <div class="box">
    <h3>Your quest:</h3>
    <p>{{ puzzle.question }}</p>

    <section>
      <button
        v-for="answer in puzzle.answers"
        :key="answer"
        @click="checkAnswer(answer)"
        :disabled="waiting || correct"
      >
        {{ answer }}
      </button>
    </section>

    <p v-if="correct" class="good">
      That's correct! Use this clue to progress further:
    </p>
    <strong v-if="correct">{{ puzzle.clue }}</strong>
    <p v-if="waiting" class="bad">
      That's an incorrect answer ;( Don't worry, you can try again in a few
      seconds!
    </p>
  </div>
</template>

<script>
  export default {
    name: "Puzzle",
    props: {
      puzzle: {
        question: String,
        solution: String,
        answers: Array,
        clue: String,
      },
    },
    data: function() {
      return {
        correct: false,
        waiting: false,
      };
    },
    methods: {
      checkAnswer: function(answer) {
        this.correct = answer === this.puzzle.solution;

        if (!this.correct) {
          this.waiting = true;
          setTimeout(() => {
            this.waiting = false;
          }, 5000);
        }
      },
    },
  };
</script>

<style scoped>
  .box {
    width: 40%;
    margin-right: 20px;
  }

  section {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  button {
    width: 48%;
    padding: 10px 0;
    margin: 10px 0;
  }

  button[disabled] {
    background: #ccc;
    cursor: default;
  }
</style>
