<template>
  <div class="box">
    <button @click="showHelpScreen()">?</button>
    <button @click="exitGame()">X</button>
  </div>
</template>

<script>
  export default {
    name: "Toolbar",
    props: {
      showHelp: Function,
      quitGame: Function,
    },
    data: function () {
      return {
        showHelpScreen: this.showHelp,
        exitGame: this.quitGame
      };
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
    border-radius: 0px 0px 0px 5px
  }

  button {
    font-size: 12px;
    font-weight: bold;
    padding: 5px 10px;
    margin: 0px 5px 5px 5px;
  }
</style>