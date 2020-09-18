<template>
  <div class="box">
    <button>?</button> <!-- TODO Show help screen -->
    <button @click="currentLevel = 0">X</button> <!-- TODO Quit game -->
  </div>
</template>

<script>
  export default {
    name: "Toolbar"
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