<template>
  <div id="app">
    <div
      class="wrapper"
      ref="wrapper"
      v-on="!flashInProcess ? { click: panelClicked } : null"
    >
      <div class="top-left-panel" ref="topLeft">
        <audio src="../src/assets/sounds/sounds_1.mp3" ref="sound1"></audio>
      </div>
      <div class="top-right-panel" ref="topRight">
        <audio src="../src/assets/sounds/sounds_2.mp3" ref="sound2"></audio>
      </div>
      <div class="bottom-left-panel" ref="bottomLeft">
        <audio src="../src/assets/sounds/sounds_3.mp3" ref="sound3"></audio>
      </div>
      <div class="bottom-right-panel" ref="bottomRight">
        <audio src="../src/assets/sounds/sounds_4.mp3" ref="sound4"></audio>
      </div>
    </div>
    <h1 v-if="!lose">Score: {{ score }} Round: {{ round }}</h1>
    <h1 v-else style="color: red; font-size: 30px">
      You lose, your score: {{ score }}
    </h1>
    <button @click="startFlashing" v-if="round === 0 && !lose">Start</button>
    <button @click="refresh" v-if="round === 0 && lose">Refresh</button>
    <div class="mode-panel">
      <h1>Choose difficulty</h1>
      <form>
        <label>
          <input
            type="radio"
            name="radio"
            value="1500"
            v-model="difficulty"
            :disabled="round > 0"
            checked
          />
          <span>easy</span>
        </label>
        <label>
          <input
            type="radio"
            name="radio"
            value="1000"
            v-model="difficulty"
            :disabled="round > 0"
          />
          <span>middle</span>
        </label>
        <label>
          <input
            type="radio"
            name="radio"
            value="400"
            v-model="difficulty"
            :disabled="round > 0"
          />
          <span>Hard</span>
        </label>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      difficulty: "1500",
      score: 0,
      sequence: [],
      lose: false,
      round: 0,
      flashInProcess: false,
    };
  },
  methods: {
    getRandomPanel() {
      const panels = this.$refs.wrapper.children;
      const panel = panels[parseInt(Math.random() * panels.length)];
      return panel;
    },
    flash(panel) {
      return new Promise((resolve) => {
        panel.classList.add("active");
        setTimeout(() => {
          panel.classList.remove("active");
          setTimeout(() => {
            resolve();
          }, 250);
        }, this.getDifficulty);
      });
    },
    async startFlashing() {
      this.flashInProcess = true;
      this.round++;
      for (let i = 0; i < this.round; i++) {
        this.sequence.push(this.getRandomPanel());
      }
      for (const panel of this.sequence) {
        panel.children[0].play();
        await this.flash(panel);
      }
      this.flashInProcess = false;
    },
    panelClicked(event) {
      if (this.round === 0) return false;
      const expectedPanel = this.sequence.shift();
      if (expectedPanel === event.target) {
        event.target.children[0].play();
        this.score++;
        if (this.sequence.length === 0) {
          this.flashInProcess = true;
          new Promise((resolve) => {
            setTimeout(() => {
              resolve();
            }, 1000);
          }).then(() => {
            this.startFlashing();
          });
        }
      } else {
        this.lose = true;
        this.round = 0;
        this.sequence = [];
      }
    },
    refresh() {
      this.score = 0;
      this.lose = false;
    },
  },
  computed: {
    getDifficulty() {
      return +this.difficulty;
    },
  },
};
</script>

<style lang="scss">
$primary-color: #00005c;
$text-color: mix(#000, $primary-color, 64%);
*,
*:after,
*:before {
  box-sizing: border-box;
}
html,
body {
  height: 100%;
}
#app {
  display: flex;
  align-items: center;
  flex-direction: column;
  height: 100%;
}
.wrapper {
  display: flex;
  width: 610px;
  flex-wrap: wrap;
}
.top-left-panel,
.top-right-panel,
.bottom-left-panel,
.bottom-right-panel {
  width: 300px;
  height: 300px;
  cursor: pointer;
  border: 2px solid black;
  &:hover {
    opacity: 0.5;
  }
  &:active {
    background: red;
  }
  &.active {
    background: white;
  }
}
.top-left-panel {
  border-top-left-radius: 300px;
  background: rgb(226, 158, 158);
}
.top-right-panel {
  border-top-right-radius: 300px;
  background: rgb(94, 250, 94);
}
.bottom-left-panel {
  border-bottom-left-radius: 300px;
  background: rgb(118, 118, 247);
}
.bottom-right-panel {
  border-bottom-right-radius: 300px;
  background: rgb(238, 252, 120);
}

form {
  display: flex;
}

label {
  display: flex;
  cursor: pointer;
  font-weight: 500;
  position: relative;
  overflow: hidden;
  margin-bottom: 0.375em;
  input {
    position: absolute;
    left: -9999px;
    &:checked + span {
      background-color: mix(#fff, $primary-color, 84%);
      &:before {
        box-shadow: inset 0 0 0 0.4375em $primary-color;
      }
    }
  }
  span {
    display: flex;
    align-items: center;
    padding: 0.375em 0.75em 0.375em 0.375em;
    border-radius: 99em; // or something higher...
    transition: 0.25s ease;
    &:hover {
      background-color: mix(#fff, $primary-color, 84%);
    }
    &:before {
      display: flex;
      flex-shrink: 0;
      content: "";
      background-color: #fff;
      width: 1.5em;
      height: 1.5em;
      border-radius: 50%;
      margin-right: 0.375em;
      transition: 0.25s ease;
      box-shadow: inset 0 0 0 0.125em $primary-color;
    }
  }
}
.mode-panel {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

button {
  margin: 10px 0;
  padding: 10px 20px;
  font-size: 35px;
  line-height: 35px;
  cursor: pointer;
  outline: none;
  border: none;
  background: orange;
  &:hover {
    color: white;
  }
}
</style>
