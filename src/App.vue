<template>
  <div id="app">
    <main>
      <!--Toolbar-->
      <Toolbar
        v-if="screen !== 'start'"
        @help="screen = 'help'"
        @exit="quitGame"
      />

      <!--Start Screen-->
      <article id="startScreen" v-if="screen === 'start'">
        <img src="@/assets/img/pubnublogo.png" alt="Game Logo" />
        <h1>Island</h1>
        <button @click="screen = 'help'">Help</button>
        <button @click="screen = 'game'">Begin!</button>
      </article>

      <!--Instructions-->
      <Help
        v-if="screen === 'help'"
        :helpText="helpText"
        @close="screen = 'game'"
      />

      <!--Main Game-->
      <article v-if="screen === 'game'">
        <Header :title="level.title" :description="level.description" />
        <section v-if="level.validator" id="gamearea">
          <Puzzle v-if="level.puzzle" :puzzle="level.puzzle" />
          <Validator v-if="level.validator" :validator="level.validator" />
        </section>
        <button
          id="previousButton"
          v-if="currentLevel > 0"
          @click="previousLevel"
        >
          &#60; Previous
        </button>
        <button
          id="nextButton"
          v-if="currentLevel < levels.length"
          :disabled="!canProgress"
          @click="nextLevel"
        >
          Next &#62;
        </button>
      </article>
    </main>
  </div>
</template>

<script>
  import Header from "./components/Header.vue";
  import Puzzle from "./components/Puzzle.vue";
  import Validator from "./components/Validator.vue";
  import Toolbar from "./components/Toolbar.vue";
  import Help from "./components/Help.vue";

  export default {
    name: "App",
    components: {
      Header,
      Puzzle,
      Toolbar,
      Help,
      Validator,
    },
    methods: {
      nextLevel: function() {
        this.currentLevel++;
        if (this.currentLevel === this.unlockLevel) {
          this.canProgress = false;
        }
      },
      previousLevel: function() {
        this.currentLevel--;
        this.canProgress = true;
      },
      handleResult: function(value) {
        if (value) {
          this.canProgress = true;
          this.unlockLevel === this.currentLevel && this.unlockLevel++;
          alert(this.level.onComplete);
        } else {
          alert("Sorry! That's a wrong answer");
        }
      },
      quitGame: function() {
        this.currentLevel = 0;
        this.canProgress = false;
        this.screen = "start";
      },
    },
    computed: {
      level: function() {
        return this.levels[this.currentLevel];
      },
    },
    data: function() {
      return {
        screen: "start",
        currentLevel: 0,
        unlockLevel: 0,
        canProgress: false,
        helpText: `<p>You awake, the lone survivor of a shipwreck on a mysterious island.
          After searching the island you discover an abandoned communications system.
          If you can figure out how to work the system you might just be able to signal for help.</p>
          <p>You've got access to the communications handbook but are missing certain configuration
          parameters. Luckily for you a suspiciously fluent parrot seems to have all the necessary
          configuration you need! Unluckily however the parrot is only willing to reveal his secrets
          if you can solve his puzzles!</p>
          <p>To escape the island you need to crack the puzzles, correctly operate the
          communications system and broadcast your SOS message!</p>`,
        levels: [
          {
            title: "Level 1: Authenticate the system",
            description: `<p>In order to use the communication system you first need to authenticate.
              Pubba the parrot will give you the authentication keys if you can solve his puzzle!<p>`,
            onComplete: `To make your own PubNub keys once you're back to safety, visit
              <a href="https://www.pubnub.com/signup" target="_blank">www.pubnub.com/signup</a> and sign up for for your own free account!`,
            puzzle: {
              question:
                "If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?",
              answers: ["1min", "5min", "2min", "10min"],
              solution: "5min",
              clue:
                "Your PubNub keyset is demo/demo. Initialize your object on the right with those keys to start working with PubNub right away!",
            },
            validator: {
              initialScript: `pubnub = new PubNub({\n  publishKey: "",\n  subscribeKey: ""\n})`,
              testHandler: (script) => {
                window.PubNub = function(obj) {
                  this.publishKey = obj.publishKey;
                  this.subscribeKey = obj.subscribeKey;
                };
                window.pubnub = {};

                eval(script);

                const result =
                  window.pubnub.publishKey === "demo" &&
                  window.pubnub.subscribeKey === "demo";
                this.handleResult(result);
              },
            },
          },
          {
            title: "Level 2: Subscribing to communication channels",
            description: `<p>Success! You've successfully authenticated yourself. It looks like this system has
            been secured with access controls however so we will need to authorize in order to broadcast our SOS.</p>
            <p>Pubba tells you that if you subscribe to the correct communication channel you can obtain the islands
            previous system operators' authorization tokens. He will only tell you the channel to listen on if you can solve his puzzle!<p>`,
            onComplete: `To learn how to subscribe to your own PubNub channels once you're back to safety,
              visit <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            puzzle: {
              question:
                "If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?",
              answers: ["1min", "5min", "2min", "10min"],
              solution: "5min",
              clue:
                "Your PubNub keyset is demo/demo. Initialize your object on the right with those keys to start working with PubNub right away!",
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';`,
              editableScript: `pubnub = new PubNub({\n  publishKey: "",\n  subscribeKey: ""\n})`,
              testHandler: (script) => {
                window.PubNub = function(obj) {
                  this.publishKey = obj.publishKey;
                  this.subscribeKey = obj.subscribeKey;
                };
                window.pubnub = {};

                eval(script);

                const result =
                  window.pubnub.publishKey === "demo" &&
                  window.pubnub.subscribeKey === "demo";
                this.handleResult(result);
              },
            },
          },
          {
            title: "Level 3: Filtering messages",
            description: `<p>Success! You can now receive inbound communications! It seems that not all the authorization
              tokens are valid however. Its possible to filter out the invalid codes if you can get Pubba to help.
              Looks like its time for another puzzle!<p>`,
            onComplete: `To learn how to use filter streaming once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            puzzle: {
              question:
                "If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?",
              answers: ["1min", "5min", "2min", "10min"],
              solution: "5min",
              clue:
                "Your PubNub keyset is demo/demo. Initialize your object on the right with those keys to start working with PubNub right away!",
            },
            validator: {
              initialScript: `pubnub = new PubNub({\n  publishKey: "",\n  subscribeKey: ""\n})`,
              testHandler: (script) => {
                window.PubNub = function(obj) {
                  this.publishKey = obj.publishKey;
                  this.subscribeKey = obj.subscribeKey;
                };
                window.pubnub = {};

                eval(script);

                const result =
                  window.pubnub.publishKey === "demo" &&
                  window.pubnub.subscribeKey === "demo";
                this.handleResult(result);
              },
            },
          },
          {
            title: "Level 4: Authorization",
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n})`,
              editableScript: `pubnub.addListener({\n  message: function(msg) { \n    console.log(msg);\n  }\n});`,
              testHandler: () => {},
            },
          },
          {
            title: "Level 5: History",
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n})`,
              editableScript: `pubnub.addListener({\n  message: function(msg) { \n    console.log(msg);\n  }\n});`,
              testHandler: () => {},
            },
          },
          {
            title: "Level 6: Publish",
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n})`,
              editableScript: `pubnub.addListener({\n  message: function(msg) { \n    console.log(msg);\n  }\n});`,
              testHandler: () => {},
            },
          },
        ],
      };
    },
  };
</script>

<style>
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  /* Remove default margin */
  body,
  h1,
  h2,
  h3,
  h4,
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

  p {
    margin-top: 0.5rem;
    margin-bottom: 0.5rem;
  }

  body {
    background-image: url("assets/img/bg.png");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center center;
    height: 100vh;
    scroll-behavior: smooth;
    text-rendering: optimizeSpeed;
    line-height: 1.5;
    font-size: 20px;
    padding-top: 50px;
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

  #gamearea {
    display: flex;
    align-items: flex-start;
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

  button:hover {
    background-color: rgb(255 134 129);
  }

  #startScreen {
    padding-top: 30vh;
    text-align: center;
  }

  #startScreen img {
    margin: auto;
    display: block;
    max-width: 30%;
  }

  #startScreen button {
    margin: 20px 10px;
  }

  #previousButton,
  #nextButton {
    margin-top: 20px;
  }

  #nextButton {
    float: right;
    margin-top: 20px;
  }

  #nextButton:disabled {
    background-color: rgb(145 136 135);
    color: #bebebe;
    cursor: default;
  }
</style>
