<template>
  <div class="box">
    <h1>PubNub Island</h1>
    <div v-html="helpText"></div>
    <section>
      <button @click="closeHelp()">Begin!</button>
    </section>
  </div>
</template>

<script>
  export default {
    name: "Help",
    props: {
      helpText: String,
      close: Function,
    },
    data: function() {
      return {
        closeHelp: this.close,
      };
    },
  };
</script>

<style scoped>
  section {
    text-align: center;
  }

  button {
    margin: 20px auto 0;
  }
</style>
