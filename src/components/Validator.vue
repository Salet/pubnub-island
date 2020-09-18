<template>
  <div class="box">
    <h3>Use your new skills here:</h3>
    <pre v-if="validator.initialScript">{{ validator.initialScript }}</pre>
    <textarea v-model="value"></textarea>
    <button @click="validator.testHandler(value)">Check</button>
  </div>
</template>

<script>
  export default {
    name: "Validator",
    props: {
      validator: {
        initialScript: String,
        editableScript: String,
        testHandler: Function,
      },
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
</style>
