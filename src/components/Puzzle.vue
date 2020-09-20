<template>
  <div class="box">
    <h3>Pubba's puzzle:</h3>
    <div v-if="!correct">
      <p v-html="puzzle.question" />
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
    </div>

    <div v-else>
      <p class="good">
        That's correct! Use this clue to progress further:
      </p>
      <strong>{{ puzzle.clue }}</strong>
      <div style="clear: both" />
      <button
        v-if="puzzle.copyString"
        @click="handleCopy"
        id="copy"
        title="Copy answer to clipboard"
      >
        <img src="@/assets/img/copy.svg" alt="Copy answer to clipboard" />
      </button>
    </div>

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
        copyString: String,
      },
    },
    data: function() {
      return {
        correct: false,
        waiting: false,
      };
    },
    watch: {
      puzzle: function() {
        this.correct = false;
      },
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
      handleCopy: function() {
        let input = document.createElement("input");
        input.setAttribute("value", this.puzzle.copyString);
        document.body.appendChild(input);
        input.select();
        let result = document.execCommand("copy");
        document.body.removeChild(input);
        return result;
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

  #copy {
    width: auto;
    float: right;
    padding: 10px 20px 6px;
    margin-top: 15px;
  }

  #copy img {
    width: 15px;
  }
</style>
