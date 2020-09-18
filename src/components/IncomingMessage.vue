<template>
  <div class="box">
    <h3>Incoming Messages!</h3>
    <pre>{{incomingMessages}}</pre>
  </div>
</template>

<script>
  export default {
    name: "Incoming Messages",
    props: {
      incomingMessages: String,
    },
  };
</script>

<style scoped>
  .box {
    position: relative;
    margin-bottom: 20px;
  }

  pre {
    background: white;
    border: 1px solid #ccc;
    display: block;
    font-family: "Courier New", Courier, monospace;
    font-size: 1rem;
    padding: 10px;
    width: 100%;
    resize: none;
    max-height: 150px;
    overflow-y: scroll;
  }
</style>
