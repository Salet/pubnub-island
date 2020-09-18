<template>
  <div class="box">
    <h3>Help</h3>
    <p>{{helpText}}</p>
  </div>
</template>

<script>
  export default {
    name: "Help",
    props: {
      helpText: String
    }
  };
</script>

<style scoped>
  .box {
    margin-right: 20px;
    flex-grow: 3;
    margin-top: 50px;
  }
</style>
