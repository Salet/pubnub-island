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
        <IncomingMessage
          v-if="level.previousResult"
          :incomingMessages="level.previousResult"
        />
        <section v-if="level.validator" id="gamearea">
          <Puzzle v-if="level.puzzle" :puzzle="level.puzzle" />
          <Validator
            v-if="level.validator"
            :validator="level.validator"
            :result="result"
            :clue="
              result == false
                ? level.onFail
                : result == true
                ? level.onComplete
                : ''
            "
            @check="validateScript"
          />
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
  import { v4 as uuidv4 } from "uuid";
  import Header from "./components/Header.vue";
  import IncomingMessage from "./components/IncomingMessage.vue";
  import Puzzle from "./components/Puzzle.vue";
  import Validator from "./components/Validator.vue";
  import Toolbar from "./components/Toolbar.vue";
  import Help from "./components/Help.vue";
  import PubNub from "pubnub";

  export default {
    name: "App",
    components: {
      Header,
      IncomingMessage,
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
          this.result = null;
        }
      },
      previousLevel: function() {
        this.currentLevel--;
        this.canProgress = true;
        this.result = true;
      },
      quitGame: function() {
        this.currentLevel = 0;
        this.canProgress = false;
        this.screen = "start";
      },
      validateScript: function(script) {
        this.result = this.level.testHandler(script);
        if (this.result) this.enableNextLevel();
      },
      enableNextLevel: function() {
        this.canProgress = true;
        this.unlockLevel === this.currentLevel && this.unlockLevel++;
      },
    },
    computed: {
      level: function() {
        return this.levels[this.currentLevel];
      },
    },
    data: function() {
      let generateAuthToken = () => uuidv4().substring(0, 10);
      let correctAuthToken = generateAuthToken();
      let generateAuthSet = (correct, code) => {
        return {
          code,
          operatorA: generateAuthToken(),
          operatorB: correct ? correctAuthToken : generateAuthToken(),
          operatorC: generateAuthToken()
        }
      }
      let filterExpression = "LOBSTER";
      let filteredAuth = generateAuthSet(true, filterExpression);
      let allAuths = [
        generateAuthSet(false, "CRAB"),
        filteredAuth,
        generateAuthSet(false, "TUNA")
      ]
      let subscribeChannel = uuidv4();
      let historyChannel = uuidv4();
      let publishChannel = uuidv4();
      let coordinates = "37°46'56.2\"N 122°23'43.1\"W";
      let historyMessage = {dailyReport: "Weather is still hot at " + coordinates + "."};

      return {
        screen: "start",
        currentLevel: 0,
        unlockLevel: 0,
        canProgress: false,
        result: null,
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
            onComplete: `<p>Nice!</p><p>To make your own PubNub keys once you're back to safety, visit
              <a href="https://www.pubnub.com/signup" target="_blank">www.pubnub.com/signup</a> and sign up for for your own free account!</p>`,
            onFail: `Something seems wrong with your script. Make sure that the resulting object is called "pubnub", it is defined
              on window scope and has a correct set of keys.`,
            puzzle: {
              question:
                "If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?",
              answers: ["1min", "5min", "10min", "25min"],
              solution: "5min",
              clue:
                "Your PubNub keyset is demo/demo. Initialize your object with those keys to start working with PubNub right away!",
            },
            validator: {
              initialScript: `import PubNub from pubnub;`,
              editableScript: `pubnub = new PubNub({\n  publishKey: "",\n  subscribeKey: ""\n});`,
            },
            testHandler: (script) => {
              window.PubNub = function(obj) {
                this.publishKey = obj.publishKey;
                this.subscribeKey = obj.subscribeKey;
              };
              window.pubnub = {};
              eval(script);
              return (
                window.pubnub.publishKey === "demo" &&
                window.pubnub.subscribeKey === "demo"
              );
            },
          },
          {
            title: "Level 2: Subscribing to communication channels",
            description: `<p>Success! You've successfully authenticated yourself. It looks like this system has
            been secured with access controls however so we will need to authorize in order to broadcast our SOS.</p>
            <p>Pubba tells you that if you subscribe to the correct communication channel you can obtain the islands
            previous system operators' authorization tokens. He will only tell you the channel to listen on if you can solve his puzzle!<p>`,
            onComplete: `Splendid! To learn how to subscribe to your own PubNub channels once you're back to safety,
              visit <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              "Something went wrong as your listener haven't received our message. Perhaps you subscribed to a wrong channel?",
            puzzle: {
              question: `<p>There are 3 crates in front of you. One contains only coconuts, one contains only bananas.
                The third crate contains a mixture of coconuts and bananas.</p>
                <p>Someone has switched around the labels however so that now none of the crates correctly match their associated label.</p>
                <p>You can't look inside the crates but can remove one piece of fruit at a time.</p>
                <p>What is the minimum number of fruits you need to inspect in order to be able to correctly identify all three crates?</p>`,
              answers: ["1", "1 < X < 6", "5 < X < 15", "15 < X"],
              solution: "1",
              clue: `You need to subscribe to channel "${subscribeChannel}". Use this channel name in the code on the
                right to start receiving PubNub messages!`,
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n})
                \npubnub.addListener({\n  message: function(msg) {\n    // you can handle incoming messages in a listener\n  }\n});`,
              editableScript: `pubnub.subscribe({channels: ['channel_name']});`,
            },
            // this particular handler might blink with an incorrect result message for a bit even for good answers
            // that's because it always returns false and the actual result is corrected when a message is received
            testHandler: (script) => {
              const msgText = "SOS";
              window.receivedMsg = { message: "" };
              window.pubnub = new PubNub({
                publishKey: "demo",
                subscribeKey: "demo",
                uuid: "demo",
              });
              window.pubnub.addListener({
                message: (msg) => {
                  this.result = msg.message === msgText;
                  if (this.result) this.enableNextLevel();
                },
              });
              // .subscribe call comes from the code editor
              eval(script);

              window.pubnub.publish({
                channel: subscribeChannel,
                message: msgText,
              });

              return false;
            },
          },
          {
            title: "Level 3: Filtering messages",
            previousResult: JSON.stringify(allAuths, null, 2),
            description: `<p>Success! You can now receive inbound communications! It seems that not all the authorization
              tokens are valid however. It's possible to filter out the invalid codes if you can get Pubba to help.</p>
              <p>Looks like its time for another puzzle!</p>`,
            onComplete: `To learn how to use filter streaming once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail: "",
            puzzle: {
              question: `<p>Adam and Eve play rock-paper-scissors 10 times. You know that:</p>
                <ul>
                  <li>Adam uses rock three times, scissors six times, and paper once.</li>
                  <li>Eve uses rock twice, scissors four times, and paper four times.</li>
                  <li>There are no ties in all 10 games.</li>
                  <li>The order of games is unknown.</li>
                </ul>
                <p>"Who wins? By how much?</p>`,
              answers: ["Adam 6-4", "Eve 6-4", "Adam 7-3", "Eve 7-3"],
              solution: "Adam 7-3",
              clue:
                'The filter expression you need is "' +
                filterExpression +
                '". Filter PubNub messages in your code on the right with this expression!',
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n});
                \npubnub.addListener({\n  message: function(msg) {\n    receivedMsg = msg;\n  }\n});
                \npubnub.subscribe({channels: ['${subscribeChannel}']});`,
              editableScript: `pubnub.setFilterExpression("");`,
            },
            testHandler: (script) => {
              window.pubnub = {};

              window.pubnub.setFilterExpression = (expression) => {
                window.pubnub.filterExpression = expression;
              };

              eval(script);

              return window.pubnub.filterExpression === filterExpression;
            },
          },
          {
            title: "Level 4: Authorization",
            previousResult: JSON.stringify(filteredAuth, null, 2),
            puzzle: {
              question: `<p>There is an island with cannibals, who always lie, explorers, who always tell the truth and pirates,
                who can be honest or dishonest.</p>
                <p>On the island you are approached by three men. One wears blue, one wears red, and one wears green.</p>
                <p>You know that one is a cannibal, one is an explorer and the other one is a pirate.</p>
                <p>The man in red says, "I am a cannibal."</p>
                <p>The man in blue says, "I am not a pirate."</p>
                <p>The man in green says, "If you asked me, I would say that the man in red is the pirate."</p>
                <p>What are the true identities of these three men?</p>`,
              answers: ["Red: Pirate, Blue: Explorer, Green: Cannibal"],
              solution: "Red: Pirate, Blue: Explorer, Green: Cannibal",
              clue: `You need to use the authorization token for operator B. Provide the correct auth token in your code on the
                right to apply the correct authorization permissions to your PubNub client!`,
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';`,
              editableScript: `pubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo",\n  authKey: ""\n});`,
              testHandler: (script) => {
                // pubnub.grant({
                //   channels: [publishChannel],
                //   authKeys: [correctAuthToken],
                //   read: true,
                //   write: true,
                // }, (status) => {
                //   // handle status
                // });
                // pubnub.grant({
                //   channels: [historyChannel],
                //   authKeys: [correctAuthToken],
                //   read: true,
                // }, (status) => {
                //   // handle status
                // });
                window.pubnub = {};
                eval(script);
                return window.pubnub.authKey === correctAuthToken;
              },
            },
          },
          {
            title: "Level 5: History",
            puzzle: {
              question: `<p>Susan and Lisa decided to play volleyball against each other.</p>
                <p>They bet 1 coconut on each game they played.</p>
                <p>Susan won three bets and Lisa walked away with 5 more coconuts than when they had started.</p>
                <p>How many games did they play?</p>`,
              answers: ["5", "8", "11", "15"],
              solution: "11",
              clue:
                'The channel you need to retrieve history from is channel "' +
                historyChannel +
                '". Fetch historical PubNub messages in your code on the right using this channel name!',
            },
            validator: {
              initialScript:
                `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n  authKey: "` +
                correctAuthToken +
                `"\n})`,
              editableScript: `pubnub.fetchMessages({\n  channels: [""], count: 1\n});`,
              testHandler: () => {},
            },
          },
          {
            title: "Level 6: Publish",
            previousResult: JSON.stringify(historyMessage, null, 2),
            puzzle: {
              question: `<p>There are five houses sitting next to each other on a neighborhood street.
                Each house's owner is of a different nationality. Each house has different colored walls.
                Each house's owner drinks their own specific beverage, smokes their own brand of cigar,
                and keeps a certain type of pet. None of the houses share any of these variables—nationality,
                wall color, beverage, cigar, and pet—they are all unique.</p><ul>
                <ul>
                  <li>The Englishman lives in the house with red walls.The Swede keeps dogs.</li>
                  <li>The Dane drinks tea.</li>
                  <li>The house with green walls is just to the left of the house with white walls.</li>
                  <li>The owner of the house with green walls drinks coffee.</li>
                  <li>The man who smokes Pall Mall keeps birds.</li>
                  <li>The owner of the house with yellow walls smokes Dunhills.</li>
                  <li>The man in the center house drinks milk.</li>
                  <li>The Norwegian lives in the first house.</li>
                  <li>The Blend smoker has a neighbor who keeps cats.</li>
                  <li>The man who smokes Blue Masters drinks beer.</li>
                  <li>The man who keeps horses lives next to the Dunhill smoker.</li>
                  <li>The German smokes Prince.</li>
                  <li>The Norwegian lives next to the house with blue walls.</li>
                  <li>The Blend smoker has a neighbor who drinks water.</li>
                </ul>
                <p>One of the house owners keeps fish, who is it?</p>`,
              answers: ["House 1", "House 2", "House 3", "House 4", "House 5"],
              solution: "House 4",
              clue:
                'You need to publish your SOS to channel "' +
                publishChannel +
                '". Publish your first PubNub message containing the island coordinates to this channel in your code on the right!',
            },
            validator: {
              initialScript:
                `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n  authKey: "` +
                correctAuthToken +
                `"\n})`,
              editableScript: `pubnub.publish({\n  channel: "",\n  message: {\n    sos: "Send Help!",\n    coordinates: "" \n  }\n});`,
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
    font-size: 18px;
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
    max-width: 1400px;
    text-align: left;
  }

  .box {
    background: rgba(255, 255, 255, 0.7);
    border-radius: 5px;
    padding: 20px 30px;
  }

  .good {
    color: green;
  }

  .bad {
    color: rgb(229, 85, 78);
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

  button[disabled] {
    cursor: not-allowed;
    background-color: #bbb;
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
    margin: 20px 0 50px;
  }

  #nextButton {
    float: right;
  }
</style>
