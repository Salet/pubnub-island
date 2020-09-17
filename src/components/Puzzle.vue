<template>
  <div class="box">
    <h3>Your quest:</h3>
  </div>
</template>

<script>
  export default {
    name: "Puzzle",
    props: {
      msg: String,
    },
  };
</script>

<style scoped>
  .box {
    margin-right: 20px;
    flex-grow: 3;
  }
</style>
