<template>
  <div class="box">
    <h1>PubNub Castaway Island</h1>
    <div v-html="helpText" />
    <section>
      <button @click="handleCloseClick">
        {{ gameStarted ? "Return to game" : "Start game!" }}
      </button>
    </section>
  </div>
</template>

<script>
  export default {
    name: "Help",
    props: {
      helpText: String,
      gameStarted: Boolean,
    },
    methods: {
      handleCloseClick: function() {
        this.$emit("close");
      },
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
