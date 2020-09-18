<template>
  <div class="box">
    <h3>Use your new skills here:</h3>
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
        testHandler: Function,
      },
    },
    data: function() {
      return {
        value: this.validator.initialScript,
      };
    },
  };
</script>

<style scoped>
  .box {
    width: 60%;
  }

  textarea {
    font-family: "Courier New", Courier, monospace;
    display: block;
    width: 100%;
    height: 250px;
    padding: 10px;
    resize: none;
    border-color: #ccc;
  }
</style>
