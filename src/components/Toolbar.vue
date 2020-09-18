<template>
  <div class="box">
    <button @click="handleHelp">?</button>
    <button @click="handleExit">X</button>
  </div>
</template>

<script>
  export default {
    name: "Toolbar",
    methods: {
      handleHelp: function() {
        this.$emit("help");
      },
      handleExit: function() {
        this.$emit("exit");
      },
    },
  };
</script>

<style scoped>
  .box {
    max-width: 20%;
    right: 0;
    position: absolute;
    top: 0;
    padding: 5px;
    border-radius: 0px 0px 0px 5px;
  }

  button {
    font-size: 12px;
    font-weight: bold;
    padding: 5px 10px;
    margin: 0px 5px 5px 5px;
  }
</style>
