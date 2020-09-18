<template>
  <div class="box">
    <h3>PubNub Island</h3>
    <div v-html="helpText"></div>

    <button @click="closeHelp()">Close</button>
  </div>
</template>

<script>
  export default {
    name: "Help",
    props: {
      helpText: String,
      close: Function
    },
    data: function () {
      return {
        closeHelp: this.close,
      };
    },
  };
</script>

<style scoped>
  .box {
    margin-right: 20px;
    flex-grow: 3;
    margin-top: 50px;
  }
</style>
