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
        <img src="@/assets/img/logo.png" alt="Game Logo" />
        <button @click="screen = 'help'">Help</button>
        <button @click="screen = 'game'">Begin!</button>
      </article>

      <!--Instructions-->
      <Help
        v-if="screen === 'help'"
        :helpText="helpText"
        :gameStarted="currentLevel > 0"
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
              result === false
                ? level.onFail
                : result === true
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

      <!--End Screen-->
      <article id="endScreen" v-if="screen === 'end'">
        <img src="@/assets/img/logo.png" alt="Game Logo" />
        <h3>You successfully sent your SOS. Help is on the way!</h3>
        <h3>
          Fancy something a little easier? Get started with your
          <a href="https://www.pubnub.com/docs">first PubNub app!</a>
        </h3>
        <button @click="quitGame">Play again?</button>
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
  // import PubNub from "pubnub";

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
        if (this.currentLevel === this.levels.length) {
          this.screen = "end";
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
        this.unlockLevel = 0;
        this.screen = "start";
      },
      validateScript: function(script) {
        try {
          this.result = this.level.testHandler(script);
        } catch (e) {
          this.result = false;
          console.log("Player code caused an error", e);
        }
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
          meta: {
            code,
          },
          message: {
            operatorA: generateAuthToken(),
            operatorB: correct ? correctAuthToken : generateAuthToken(),
            operatorC: generateAuthToken(),
          },
        };
      };
      let filterExpression = "LOBSTER";
      let filteredAuth = generateAuthSet(true, filterExpression);
      let allAuths = [
        generateAuthSet(false, "CRAB"),
        filteredAuth,
        generateAuthSet(false, "TUNA"),
      ];
      let subscribeChannel = uuidv4();
      let historyChannel = uuidv4();
      let publishChannel = uuidv4();
      let coordinates = "37°46'56.2\"N 122°23'43.1\"W";
      let historyMessage = {
        dailyReport:
          "Weather is still hot at " + coordinates + " sun protection advised!",
      };

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
            description: `<p>In order to use the communication system you first need to authenticate yourself.
              Pubba the parrot will give you the authentication keys if you can solve his puzzle!</p>`,
            onComplete: `<p>Nice! To make your own PubNub keys once you're back to safety, visit
              <a href="https://www.pubnub.com/signup" target="_blank">www.pubnub.com/signup</a> and sign up for for your own free account!</p>`,
            onFail: `Hmmm, something seems wrong with your script. Use the code template and the answer to the puzzle to
             correctly define your PubNub object with the correct set of keys.`,
            puzzle: {
              question:
                "If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?",
              answers: ["1min", "5min", "10min", "25min"],
              solution: "5min",
              clue:
                "Your PubNub keyset is demo/demo. Initialize your object with those keys to start working with PubNub right away!",
              copyString: "demo",
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
            description: `<p>Success! You've successfully authenticated yourself! It looks like this system has
            been secured with access controls however. You'll need authorization in order to broadcast the SOS.</p>
            <p>Pubba tells you that if you subscribe to the correct communication channel you can obtain the authorization tokens of the old operators.
             He will only tell you the channel to listen on if you can solve his puzzle however!</p>`,
            onComplete: `Splendid! To learn how to subscribe to your own PubNub channels once you're back to safety,
              visit <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              "Hmm..that doesn't look right. Perhaps you subscribed to the wrong channel?",
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
              copyString: subscribeChannel,
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n})
                \npubnub.addListener({\n  message: function(msg) {\n    // you can handle incoming messages in a listener\n  }\n});`,
              editableScript: `pubnub.subscribe({channels: [""]});`,
            },
            // this particular handler might blink with an incorrect result message for a bit even for good answers
            // that's because it always returns false and the actual result is corrected when a message is received
            testHandler: (script) => {
              // const msgText = "SOS";
              // window.receivedMsg = { message: "" };
              // window.pubnub = new PubNub({
              //   publishKey: "demo",
              //   subscribeKey: "demo",
              //   uuid: "demo",
              // });
              // window.pubnub.addListener({
              //   message: (msg) => {
              //     this.result = msg.message === msgText;
              //     if (this.result) this.enableNextLevel();
              //   },
              // });
              // // .subscribe call comes from the code editor
              // eval(script);
              //
              // window.pubnub.publish({
              //   channel: subscribeChannel,
              //   message: msgText,
              // });
              window.pubnub = {
                subscribe: (obj) => {
                  window.subscribeChannels = obj.channels;
                },
              };

              eval(script);

              return (
                window.subscribeChannels.length === 1 &&
                window.subscribeChannels[0] === subscribeChannel
              );
            },
          },
          {
            title: "Level 3: Filtering messages",
            previousResult: JSON.stringify(allAuths, null, 2),
            description: `<p>Awesome! You can now receive inbound communications and have intercepted messages containing authorization tokens.</p>
              <p>It seems that not all the authorization tokens are valid however. The invalid codes can be filtered out but you'll need Pubba to tell you which ones to keep!</p>
              <p>Looks like its time for another puzzle!</p>`,
            onComplete: `Woo! To learn how to use PubNub's stream filtering once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              'Hmm that doesn\'t look right. Your filter expression should be of the form "code == ANIMAL"',
            puzzle: {
              question: `<p>Adam and Eve play rock-paper-scissors 10 times. You know that:</p>
                <ul>
                  <li>Adam uses rock three times, scissors six times, and paper once.</li>
                  <li>Eve uses rock twice, scissors four times, and paper four times.</li>
                  <li>There are no ties in all 10 games.</li>
                  <li>The order of games is unknown.</li>
                </ul>
                <p>Who wins and by how much?</p>`,
              answers: ["Adam 6-4", "Eve 6-4", "Adam 7-3", "Eve 7-3"],
              solution: "Adam 7-3",
              clue: `The filter expression you need is "${filterExpression}". Filter PubNub messages in your code on the right with this expression!`,
              copyString: filterExpression,
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n});
                \npubnub.addListener({\n  message: function(msg) {\n    receivedMsg = msg;\n  }\n});
                \npubnub.subscribe({channels: ["${subscribeChannel}"]});`,
              editableScript: `pubnub.setFilterExpression("code == ");`,
            },
            testHandler: (script) => {
              window.pubnub = {
                setFilterExpression: (expression) => {
                  window.pubnub.filterExpression = expression;
                },
              };

              eval(script);

              return (
                window.pubnub.filterExpression === "code == " + filterExpression
              );
            },
          },
          {
            title: "Level 4: Authorization",
            description: `<p>Excellent! You've filtered out all the invalid operator codes!</p>
              <p>You need to go one step further as only one of the operator codes gives you permission to publish messages.</p>
              <p>You'll need to solve another puzzle to determine which operator code to use!</p>`,
            onComplete: `Great job! To learn how to use PubNub's Access Manager once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              "Hmm that doesn't look right. Only one of the operator's auth tokens are valid. Solve the puzzle to work out which!",
            previousResult: JSON.stringify(filteredAuth, null, 2),
            puzzle: {
              question: `<p>There is an island with cannibals, who always lie, explorers, who always tell the truth and pirates,
                who can be honest or dishonest.</p>
                <p>On the island you are approached by three men. One wears blue, one wears red, and one wears green.</p>
                <ul>
                  <li>You know that one is a cannibal, one is an explorer and the other one is a pirate.</li>
                  <li>The man in red says, "I am a cannibal."</li>
                  <li>The man in blue says, "I am not a pirate."</li>
                  <li>The man in green says, "If you asked me, I would say that the man in red is the pirate."</li>
                </ul>
                <p>What are the true identities of these three men?</p>`,
              answers: [
                "Red: Pirate, Blue: Explorer, Green: Cannibal",
                "Red: Explorer, Blue: Cannibal, Green: Pirate",
                "Red: Cannibal, Blue: Pirate, Green: Explorer",
                "Red: Pirate, Blue: Cannibal, Green: Explorer",
              ],
              solution: "Red: Pirate, Blue: Explorer, Green: Cannibal",
              clue: `You need to use the authorization token for operator B. Provide the correct auth token in your code on the
                right to apply the correct authorization permissions to your PubNub client!`,
              copyString: filteredAuth.message.operatorB,
            },
            validator: {
              initialScript: `import PubNub from 'pubnub';`,
              editableScript: `pubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo",\n  authKey: ""\n});`,
            },
            testHandler: (script) => {
              window.PubNub = function(obj) {
                this.authKey = obj.authKey;
              };
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
          {
            title: "Level 5: History",
            description: `<p>Sweet! It looks like you finally have full access to the system, but before you can send your SOS message, you need to know the island's coordinates.</p>
              <p>According to Pubba, the island operators used to send weather updates which included the island's coordinates. All of the historical messages have been stored and should be retrievable.</p>
              <p>If you are able to solve the next puzzle, Pubba will let you know the channel name they used.</p>`,
            onComplete: `Good work! To learn how to PubNub's history features once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              "Hmm that doesn't look right. You need to fetch just one message from the correct channel!",
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
              copyString: historyChannel,
            },
            validator: {
              initialScript:
                `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n  authKey: "` +
                correctAuthToken +
                `"\n})`,
              editableScript: `pubnub.fetchMessages({\n  channels: [""],\n  count: 1\n});`,
            },
            testHandler: (script) => {
              window.pubnub = {
                fetchMessages: (obj) => {
                  window.fetchChannels = obj.channels;
                  window.fetchCount = obj.count;
                },
              };
              eval(script);
              return (
                window.fetchChannels.length === 1 &&
                window.fetchChannels[0] === historyChannel &&
                window.fetchCount === 1
              );
            },
          },
          {
            title: "Level 6: Publish",
            description: `<p>Fantastic! Thanks to Pubba's knowledge and your superb puzzle solving skills, you now have full access to the system and know the island coordinates!</p>
              <p>In order to broadcast your SOS you just need to know the channel on which to publish your message to.</p>
              <p>It looks like you'll need Pubba to help one last time. This puzzle isn't going to be easy!</p>`,
            onComplete: `Superb! To learn more about to PubNub's realtime publish and subscribe functionality once you're back to safety, visit
              <a href="https://www.pubnub.com/docs" target="_blank">www.pubnub.com/docs</a>!`,
            onFail:
              "Hmm that doesn't look right. You need to publish the island's coordinates to the correct channel. Solve the puzzle and check your formatting!",
            previousResult: JSON.stringify(historyMessage, null, 2),
            puzzle: {
              question: `<p>There are five houses sitting next to each other on a neighborhood street.
                Each house's owner is of a different nationality. Each house has different colored walls.
                Each house's owner drinks their own specific beverage, smokes their own brand of cigar,
                and keeps a certain type of pet. None of the houses share any of these variables—nationality,
                wall color, beverage, cigar, and pet—they are all unique.</p>
                <ul>
                  <li>The Englishman lives in the house with red walls.</li>
                  <li>The Swede keeps dogs.</li>
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
              copyString: publishChannel,
            },
            validator: {
              initialScript:
                `import PubNub from 'pubnub';\n\npubnub = new PubNub({\n  publishKey: "demo",\n  subscribeKey: "demo"\n  authKey: "` +
                correctAuthToken +
                `"\n})`,
              editableScript: `pubnub.publish({\n  channel: "",\n  message: {\n    sos: "Send help!",\n    coordinates: "" \n  }\n});`,
            },
            testHandler: (script) => {
              window.pubnub = {
                publish: (obj) => {
                  window.publishChannel = obj.channel;
                  window.publishMessage = obj.message;
                },
              };
              eval(script);
              return (
                window.publishChannel === publishChannel &&
                window.publishMessage &&
                window.publishMessage.sos === "Send help!" &&
                window.publishMessage.coordinates === coordinates
              );
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
    background-image: url("assets/img/background.png");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center center;
    background-attachment: fixed;
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

  a {
    color: #2c3e50;
  }

  main {
    margin: 0 auto;
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
    margin-top: 0;
  }

  button:hover {
    background-color: rgb(255 134 129);
  }

  button[disabled] {
    cursor: not-allowed;
    background-color: #bbb;
  }

  #startScreen,
  #endScreen {
    padding-top: 20vh;
    text-align: center;
  }

  #startScreen img,
  #endScreen img {
    margin: 0 auto 35px;
    display: block;
    max-width: 30%;
  }

  #startScreen button,
  #endScreen button {
    margin: 20px 10px;
  }

  #endScreen img {
    margin: 0 auto 10px;
  }

  #endScreen h3 {
    font-weight: 900;
  }

  #endScreen button {
    margin: 35px 10px 0;
  }

  #previousButton,
  #nextButton {
    margin: 20px 0 50px;
  }

  #nextButton {
    float: right;
  }
</style>
