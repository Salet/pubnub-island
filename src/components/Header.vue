<template>
  <div class="box">
    <img src="@/assets/img/parrot.svg" />
    <h1>{{ title }}</h1>
    <div v-html="description" />
  </div>
</template>

<script>
  export default {
    name: "Header",
    props: {
      title: String,
      description: String,
    },
  };
</script>

<style scoped>
  .box {
    padding-left: 120px;
    position: relative;
    margin-bottom: 20px;
  }

  img {
    position: absolute;
    top: -12px;
    left: 0;
    display: inline-block;
    width: 100px;
    vertical-align: center;
  }

  h1 {
    display: inline;
    vertical-align: center;
  }
</style>
