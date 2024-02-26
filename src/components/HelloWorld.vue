<template>
  <div>
    <div>round {{ round }}</div>
    <div class="game-board">
      <button v-for="(button, index) in buttons" @click="clickButton(index)" :key="index"
        :class="['game-button', button.color, button.active ? 'active' : '', isPlaying ? 'playing' : '']"></button>
    </div>
    <div class="levels">
      <span>Change level</span>

      <label>
        <input v-model="level" value="1" name="level" type="radio" />
        <span>easy</span>
      </label>

      <label>
        <input v-model="level" value="2" name="level" type="radio" />
        <span>medium</span>
      </label>

      <label>
        <input v-model="level" value="3" name="level" type="radio" />
        <span>hard</span>
      </label>
    </div>
    <div class="game-controls">
      <button @click="startRound">Start</button>
      <button @click="resetGame">Reset</button>
    </div>
  </div>
</template>

<script>
import sound1 from "../assets/1.ogg"
import sound2 from "../assets/2.ogg"
import sound3 from "../assets/3.ogg"
import sound4 from "../assets/4.ogg"


const audio1 = new Audio()



export default {
  data() {
    return {
      buttons: [
        { id: 0, color: 'red', active: false, sound: sound1 },
        { id: 1, color: 'blue', active: false, sound: sound2 },
        { id: 2, color: 'green', active: false, sound: sound3 },
        { id: 3, color: 'yellow', active: false, sound: sound4 },
      ],
      sequence: [],
      round: 1,
      isPlaying: false,
      level: 1,
    };
  },
  methods: {
    resetGame() {
      this.playerSequence = [];
      this.sequence = [];
      this.round = 1;
      this.isPlaying = false
    },

    startRound() {
      let i = 0;
      this.sequence = []
      const value = Math.floor(Math.random() * 4)
      this.sequence.push(value)
      this.buttons[value].active = true
      const audio = new Audio(this.buttons[value].sound);
      const audioPlay = audio.play();
      i++;
      let timeout = setTimeout(() => {
        this.buttons[value].active = false;
        if (audioPlay !== undefined) {
          audioPlay.then(() => {
            audio.pause()
          })
        }
        clearTimeout(timeout);
      }, this.getTimeDifficulity() * 2 / 3)

      const p = new Promise((res) => {
        let timer = setInterval(() => {
          if (i < this.round) {
            const value = Math.floor(Math.random() * 4)
            this.sequence.push(value)
            this.buttons[value].active = true
            const audio = new Audio(this.buttons[value].sound);
            const audioPlay = audio.play();
            i++;
            let timeout = setTimeout(() => {
              this.buttons[value].active = false;
              if (audioPlay !== undefined) {
                audioPlay.then(() => {
                  audio.pause()
                })
              }
              clearTimeout(timeout);
            }, this.getTimeDifficulity() * 2 / 3)
          }
          else {
            clearTimeout(timer);
            res('')
          }
        }, this.getTimeDifficulity())
      })

      p.then(() => {
        this.isPlaying = true;
      })

    },
    getTimeDifficulity() {
      if (this.level == 1) {
        return 1500;
      }
      if (this.level == 2) {
        return 1000;
      }
      if (this.level == 3) {
        return 400;
      }
    },
    clickButton(index) {
      if (this.isPlaying) {
        if (!audio1.paused) {
          audio1.pause()
        }
        audio1.src = this.buttons[index].sound;
        audio1.play();

        if (this.sequence[0] === index) {
          this.sequence.shift();
        }
        else {
          this.resetGame()
          return;
        }

        if (this.sequence.length === 0) {
          this.round = this.round + 1;
          this.isPlaying = false;
          setTimeout(() => {
            this.startRound()
          }, 500)
        }
      }
    }
  }
};
</script>

<style scoped>
.game-board {
  display: grid;
  grid-template-columns: 1fr 1fr;
  width: 200px;
  gap: 0;
  margin: 0 auto;
  margin-top: 20px;

}

.game-button {
  width: 100px;
  height: 100px;
  cursor: pointer;
  border: 0;
}

.game-button.playing:hover {
  opacity: 0.5;
}

.red {
  background-color: red;
}

.blue {
  background-color: blue;
}

.green {
  background-color: green;
}

.yellow {
  background-color: yellow;
}

.levels {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin: 15px 0 0 0;
}

.active {
  opacity: 0.5;
}

.game-controls {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;

  button {
    font-size: 16px;
  }
}
</style>