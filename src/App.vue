<template>
  <div id="app">
    <h1>Simon The Game Copy</h1>
    <small>by Dashi Dantsaranov</small>
    <div id="status">
      {{ displayMessage }}
    </div>
    <div id="simon">
      <div class="row">
        <div
          id="green"
          v-bind:class="{ bright: isActive && currentLight === 'green' }"
          class="light col noSelect"
          v-on:click="captureTap('green')"
        ></div>
        <div
          id="red"
          v-bind:class="{ bright: isActive && currentLight === 'red' }"
          class="light col noSelect"
          v-on:click="captureTap('red')"
        ></div>
      </div>

      <div class="row">
        <div
          id="yellow"
          v-bind:class="{ bright: isActive && currentLight === 'yellow' }"
          class="light col noSelect"
          v-on:click="captureTap('yellow')"
        ></div>
        <div
          id="blue"
          v-bind:class="{ bright: isActive && currentLight === 'blue' }"
          class="light col noSelect"
          v-on:click="captureTap('blue')"
        ></div>
      </div>
    </div>

    <div class="flex">
      <div class="game-options">
        <h3>Game Options:</h3>
        <input
          type="radio"
          name="mode"
          value="easy"
          v-model="modeGame"
        />Easy<br />
        <input
          type="radio"
          name="mode"
          value="normal"
          v-model="modeGame"
        />Normal<br />
        <input
          type="radio"
          name="mode"
          value="hard"
          v-model="modeGame"
        />Hard<br />
      </div>

      <div id="controls" class="row">
        <div class="col">
          <button class="start" v-on:click="start">
            {{ buttonMessage }}
          </button>
        </div>
      </div>
    </div>

    <div id="history">
      <p><strong>Current Sequence:</strong> {{ current }}</p>
      <p><strong>Longest Sequence:</strong> {{ longest }}</p>
    </div>

    <div class="hide">
      <audio
        v-for="(sound, color) in sounds"
        :key="color"
        :ref="'sound' + color"
      >
        <source :src="sound" type="audio/mpeg" />
      </audio>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      longest: 0,
      sequence: ["red", "blank"],
      taps: "",
      lights: ["red", "green", "yellow", "blue"],
      isActive: false,
      currentLight: "",
      iterationCounter: 0,
      displayMessage: "",
      buttonMessage: "Start",
      disabled: true,
      speed: 400,
      modeGame: "normal",
      sounds: {
        green: "https://s3.amazonaws.com/freecodecamp/simonSound1.mp3",
        red: "https://s3.amazonaws.com/freecodecamp/simonSound2.mp3",
        yellow: "https://s3.amazonaws.com/freecodecamp/simonSound3.mp3",
        blue: "https://s3.amazonaws.com/freecodecamp/simonSound4.mp3",
      },
    };
  },
  computed: {
    current: function () {
      return this.sequence.length / 2;
    },
  },

  mounted() {},

  methods: {
    start: function () {
      this.sequence = [];
      this.addToSequence();
      this.playSequence(this.sequence);
      this.startTimer();
      this.iterationCounter = 0;
      this.displayMessage = "";
      this.speed = 400;
    },
    playSound(color) {
      if (this.$refs["sound" + color] !== undefined) {
        this.$refs["sound" + color][0].play();
      }
    },
    chooseRandomLight: function () {
      var index = Math.floor(Math.random() * 4);
      return this.lights[index];
    },
    addToSequence: function () {
      this.sequence.push(this.chooseRandomLight(), "blank");
      this.buttonMessage = "Restart";
    },
    playSequence: function (lightSelected) {
      clearTimeout();
      if (this.modeGame === "easy") {
        this.speed = 1500;
        if (this.modeGame === "normal") {
          this.speed = 1000;
          if (this.modeGame === "hard") {
            this.speed = 400;
          }
        }
      }
      var self = this;
      lightSelected.forEach(function (color, index) {
        setTimeout(function () {
          self.changeLightState(index);
          self.playSound(color);
        }, index * self.speed);
        self.isActive = false;
      });
    },
    changeLightState: function (iteration) {
      var self = this;
      this.isActive = true;
      this.currentLight = self.sequence[iteration];
    },
    captureTap: function (color, time) {
      this.playSound(color);
      this.taps = color;
      this.currentLight = color;
      setTimeout(
        function () {
          this.gameEvaluation(color);
        }.bind(this),
        200
      );
    },
    gameEvaluation: function (color) {
      if (this.sequence[this.iterationCounter] === this.taps) {
        this.iterationCounter += 2;
        this.currentLight = "blank";
        clearTimeout();
        if (this.iterationCounter === this.sequence.length) {
          this.currentLight = false;
          this.addToSequence();
          this.displayMessage = `Round: ${this.current}`;
          setTimeout(
            function () {
              this.playSequence(this.sequence);
              this.displayMessage = "";
            }.bind(this),
            2000
          );
          this.startTimer();
          this.iterationCounter = 0;
        }
      } else {
        if (this.sequence.length / 2 > this.longest) {
          this.currentLight = false;
          this.longest = this.sequence.length / 2 - 1;
          this.displayMessage =
            "Sorry, not quite right. But hey, this is your longest sequence yet!";
          this.buttonMessage = "Try Again";
          clearTimeout();
        } else {
          this.currentLight = false;
          this.displayMessage =
            "Sorry, not quite right. But you're close, give it another try.";
          this.buttonMessage = "Try Again";
        }
      }
    },
    startTimer: function () {},
  },
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
h1 {
  margin-top: 30px;
  margin-bottom: 5px;
}
#simon {
  background-color: white;
  max-width: 480px;
  margin: 0 auto;
  padding: 10px;
}
#status {
  margin-top: 30px;
  margin-bottom: 15px;
  height: 10px;
  font-size: 20px;
}
.col {
  display: inline-block;
}
.start {
  padding: 10px;
  font-size: 18px;
}
.light {
  margin: 10px;
  border: 1px solid #000;
}
#red {
  height: 150px;
  width: 170px;
  background-color: red;
  background-size: 170px 150px;
  opacity: 0.3;
}
#yellow {
  height: 150px;
  width: 170px;
  background-color: yellow;
  background-size: 170px 150px;
  opacity: 0.3;
}
#green {
  height: 150px;
  width: 170px;
  background-color: green;
  background-size: 170px 150px;
  opacity: 0.3;
}
#blue {
  height: 150px;
  width: 170px;
  background-color: blue;
  background-size: 170px 150px;
  opacity: 0.3;
}
.bright {
  opacity: 1 !important;
}
.flex {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}
.game-options {
  border: 1px solid #000;
  margin-right: 15px;
  padding: 15px;
  text-align: left;
}
.game-options h3 {
  margin-top: 0;
  margin-bottom: 0;
}
.game-options input[type="radio"] {
  margin-right: 10px;
}
#history {
  margin-top: 25px;
}
.noSelect {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
</style>