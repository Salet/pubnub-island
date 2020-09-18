<template>
  <div id="app">
    <main>
      <!--Toolbar-->
      <Toolbar v-if="screen !== 'start'"/>

      <!--Start Screen-->
      <article id="startScreen" v-if="screen === 'start'">
        <img src="@/assets/img/pubnublogo.png">
        <button @click="screen = 'help'">Instructions</button>
        <button @click="screen = 'game'">Begin!</button>
      </article>

      <!--Instructions-->
      <Help v-if="screen === 'help'"
        :helpText="helpText"
      />

      <!--Main Game-->
      <article v-if="screen === 'game'">
        <Header
          :title="levels[currentLevel].title"
          :description="levels[currentLevel].description"
        />
        <section>
          <Puzzle />
          <Answer />
        </section>
        <button id="nextButton" @click="[currentLevel++]">Next ></button>
      </article>
    </main>
  </div>
</template>

<script>
  import Header from "./components/Header.vue";
  import Puzzle from "./components/Puzzle.vue";
  import Answer from "./components/Answer.vue";
  import Toolbar from "./components/Toolbar.vue";
  import Help from "./components/Help.vue";

  export default {
    name: "App",
    components: {
      Header,
      Puzzle,
      Answer,
      Toolbar,
      Help
    },
    data: function() {
      return {
        screen: "start",
        currentLevel: 0,
        helpText: `You awake, the lone survivor of a shipwreck on a mysterious island. After searching the island you discover a now abandoned communications system. If you can figure out how to work the system you just might be able to signal for help.

         You've got access to the communications handbook but are missing certain configuration parameters. Luckily it seems the old system operators have hidden the necessary configuration through a series of puzzles.

         To escape the island you need to crack the puzzles, correctly operate the communications system and broadcast your SOS message!`,
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
    background-image: url("assets/img/bg.png");
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
    margin: 0px auto;
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
    margin-top: 0px;
  }

  #startScreen {
    text-align: center;
  }

  #startScreen img {
    margin: auto;
    display: block;
    max-width: 50%;
  }

  #startScreen button {
    margin: 20px 10px
  }

  #nextButton {
    float: right;
  }
</style>