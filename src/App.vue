<template>
  <div id="app">
    <main>
      <Header
        :title="levels[currentLevel].title"
        :description="levels[currentLevel].description"
      />
      <!-- <section v-if="levels[currentLevel].puzzle"> -->
      <section>
        <Puzzle />
        <Answer />
      </section>
      <button @click="currentLevel++">Next ></button>
    </main>
  </div>
</template>

<script>
  import Header from "./components/Header.vue";
  import Puzzle from "./components/Puzzle.vue";
  import Answer from "./components/Answer.vue";

  export default {
    name: "App",
    components: {
      Header,
      Puzzle,
      Answer,
    },
    data: function() {
      return {
        currentLevel: 0,
        levels: [
          {
            title: "Welcome to PubNub Island!",
            description: `Hello. It looks like you landed on a deserted island of cross-device communication. Traditionally, one of the ways to signal your need of help would be through smoke signals.

              Fortunately, nowadays there's a system that might help more easily and with better results: PubNub!

              Lets quickly learn how to communicate with other people and devices through the use of PubNub.`,
          },
          {
            title: "Level 1: Getting your keys!",
          },
        ],
      };
    },
  };
</script>

<style>
  /* Box sizing rules */
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  /* Remove default padding */
  ul[class],
  ol[class] {
    padding: 0;
  }

  /* Remove default margin */
  body,
  h1,
  h2,
  h3,
  h4,
  p,
  ul[class],
  ol[class],
  li,
  figure,
  figcaption,
  blockquote,
  dl,
  dd {
    margin: 0;
  }

  body {
    background-image: url("assets/bg.png");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center center;
    min-height: 100vh;
    scroll-behavior: smooth;
    text-rendering: optimizeSpeed;
    line-height: 1.5;
    font-size: 20px;
  }

  ul[class],
  ol[class] {
    list-style: none;
  }

  img {
    max-width: 100%;
  }

  input,
  button,
  textarea,
  select {
    font: inherit;
  }

  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    height: 100vh;
    text-align: center;
  }

  main {
    margin: 50px auto;
    max-width: 1200px;
    text-align: left;
  }

  .box {
    background: rgba(255, 255, 255, 0.7);
    border-radius: 5px;
    padding: 20px 30px;
  }

  section {
    display: flex;
  }

  button {
    background-color: rgb(229, 85, 78);
    border: none;
    border-radius: 5px;
    color: white;
    cursor: pointer;
    padding: 20px 30px;
    margin-top: 20px;
    float: right;
  }
</style>
