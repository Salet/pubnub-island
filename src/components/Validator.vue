<template>
  <div class="box">
    <h3>Use the clue here:</h3>
    <pre v-if="validator.initialScript">{{ validator.initialScript }}</pre>
    <textarea v-model="value"></textarea>
    <div id="clue" v-html="clue" :class="result ? 'good' : 'bad'"></div>
    <button v-if="!result" @click="handleCheck">Check</button>
  </div>
</template>

<script>
  export default {
    name: "Validator",
    props: {
      validator: {
        initialScript: String,
        editableScript: String,
      },
      result: Boolean,
      clue: String,
    },
    watch: {
      validator: function(newValidator) {
        this.value = newValidator.editableScript;
      },
    },
    data: function() {
      return {
        value: this.validator.editableScript,
      };
    },
    methods: {
      handleCheck: function() {
        this.$emit("check", this.value);
      },
    },
  };
</script>

<style scoped>
  .box {
    width: 60%;
  }

  pre,
  textarea {
    background: white;
    border: 1px solid #ccc;
    display: block;
    font-family: "Courier New", Courier, monospace;
    font-size: 1rem;
    padding: 10px;
    width: 100%;
  }

  pre {
    margin-bottom: 0;
    border-bottom: none;
  }

  textarea {
    height: 250px;
    resize: none;
  }

  button {
    margin-top: 20px;
    float: right;
  }

  #clue {
    margin-top: 20px;
  }
</style>
