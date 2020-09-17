<template>
  <div class="box">
    <h3>Your result:</h3>
  </div>
</template>

<script>
  export default {
    name: "Answer",
    props: {
      msg: String,
    },
  };
</script>

<style scoped>
  .box {
    flex-grow: 7;
  }
</style>
