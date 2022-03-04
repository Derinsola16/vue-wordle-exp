<template>
  <div class="hello">
    <Transition>
    <div class="message" v-if="message">
      {{ message }}
      <pre v-if="grid">{{ grid }}</pre>
    </div>
  </Transition>
    <div id="board">
      <div
        v-for="(row, index) in board"
        :key="index"
        :class="[
          'row',
          shakeRowIndex === index && 'shake',
          success && currentRowIndex === index && 'jump',
        ]"
      >
        <div
          v-for="(tile, index) in row"
          :key="index"
          :class="['tile', tile.letter && 'filled', tile.state && 'revealed']"
        >
          <div class="front" :style="{ transitionDelay: `${index * 300}ms` }">
            {{ tile.letter }}
          </div>
          <div
            :class="['back', tile.state]"
            :style="{
              transitionDelay: `${index * 300}ms`,
              animationDelay: `${index * 100}ms`,
            }"
          >
            {{ tile.letter }}
          </div>
        </div>
      </div>
    </div>
    <Keyboard @key="onKey" :letter-states="letterStates" />
  </div>
</template>

<script>
import { getDayWord, allWords } from "./words";
import { State } from "./type";
import Keyboard from "./keyboard";
export default {
  name: "HelloWorld",
  components: {
    Keyboard,
  },
  data() {
    return {
      answer: "",
      icons: {
        [State.CORRECT]: "🟩",
        [State.PRESENT]: "🟨",
        [State.ABSENT]: "⬜",
        [State.INITIAL]: null,
      },
      board: null,
      currentRowIndex: 0,
      message: "",
      grid: "",
      shakeRowIndex: -1,
      success: false,
      letterStates: {},
      allowInput: true,
    };
  },

  methods: {
    onKeyup(e) {
      this.onKey(e.key);
    },

    onKey(key) {
      if (!this.allowInput) return;
      if (/^[a-zA-Z]$/.test(key)) {
        this.fillTile(key.toLowerCase());
      } else if (key === "Backspace") {
        this.clearTile();
      } else if (key === "Enter") {
        this.completeRow();
      }
    },

    fillTile(letter) {
      for (const tile of this.currentRow) {
        if (!tile.letter) {
          tile.letter = letter;
          break;
        }
      }
    },

    clearTile() {
      for (const tile of [...this.currentRow].reverse()) {
        if (tile.letter) {
          tile.letter = "";
          break;
        }
      }
    },

    completeRow() {
      if (this.currentRow.every((tile) => tile.letter)) {
        const guess = this.currentRow.map((tile) => tile.letter).join("");
        if (!allWords.includes(guess) && guess !== this.answer) {
          this.shake();
          this.showMessage(`Not in word list`);
          return;
        }
        const answerLetters = this.answer.split("");
        // first pass: mark correct ones
        this.currentRow.forEach((tile, i) => {
          if (answerLetters[i] === tile.letter) {
            tile.state = this.letterStates[tile.letter] = State.CORRECT;
            answerLetters[i] = null;
          }
        });
        // second pass: mark the present
        this.currentRow.forEach((tile) => {
          if (!tile.state && answerLetters.includes(tile.letter)) {
            tile.state = State.PRESENT;
            answerLetters[answerLetters.indexOf(tile.letter)] = null;
            if (!this.letterStates[tile.letter]) {
              this.letterStates[tile.letter] = State.PRESENT;
            }
          }
        });
        // 3rd pass: mark absent
        this.currentRow.forEach((tile) => {
          if (!tile.state) {
            tile.state = State.ABSENT;
            if (!this.letterStates[tile.letter]) {
              this.letterStates[tile.letter] = State.ABSENT;
            }
          }
        });
        this.allowInput = false;
        if (this.currentRow.every((tile) => tile.state === State.CORRECT)) {
          // yay!
          setTimeout(() => {
            this.grid = this.genResultGrid();
            this.showMessage(
              [
                "Genius",
                "Magnificent",
                "Impressive",
                "Splendid",
                "Great",
                "Phew",
              ][this.Index],
              -1
            );
            this.success = true;
          }, 1600);
        } else if (this.currentRowIndex < this.board.length - 1) {
          // go the next row
          this.currentRowIndex++;
          setTimeout(() => {
            this.allowInput = true;
          }, 1600);
        } else {
          // game over :(
          setTimeout(() => {
            this.showMessage(this.answer.toUpperCase(), -1);
          }, 1600);
        }
      } else {
        this.shake();
        this.showMessage("Not enough letters");
      }
    },

    showMessage(msg, time = 1000) {
      this.message = msg;
      if (time > 0) {
        setTimeout(() => {
          this.message = "";
        }, time);
      }
    },

    shake() {
      this.shakeRowIndex = this.currentRowIndex;
      setTimeout(() => {
        this.shakeRowIndex = -1;
      }, 1000);
    },

    genResultGrid() {
      return this.board
        .slice(0, this.currentRowIndex + 1)
        .map((row) => {
          return row.map((tile) => this.icons[tile.state]).join("");
        })
        .join("\n");
    },
  },

  computed: {
    currentRow() {
      return this.board[this.currentRowIndex];
    },
  },

  mounted() {
    const word = getDayWord();
    this.answer = word;

   
    this.board = Array.from({ length: 6 }, () =>
      Array.from({ length: 5 }, () => ({
        letter: "",
        state: State.INITIAL,
      }))
    );

     console.log(typeof(this.board))


    window.addEventListener("keyup", this.onKeyup);
  },

  beforeDestroy() {
    window.removeEventListener("keyup", this.onKeyup);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#board {
  display: grid;
  grid-template-rows: repeat(6, 1fr);
  grid-gap: 5px;
  padding: 10px;
  box-sizing: border-box;
  --height: min(420px, calc(var(--vh, 100vh) - 310px));
  height: var(--height);
  width: min(350px, calc(var(--height) / 6 * 5));
  margin: 0px auto;
}
.message {
  position: absolute;
  left: 50%;
  top: 80px;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.85);
  padding: 16px 20px;
  z-index: 2;
  border-radius: 4px;
  transform: translateX(-50%);
  transition: opacity 0.3s ease-out;
  font-weight: 600;
}
.message.v-leave-to {
  opacity: 0;
}
.row {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-gap: 5px;
}
.tile {
  width: 100%;
  font-size: 2rem;
  line-height: 2rem;
  font-weight: bold;
  vertical-align: middle;
  text-transform: uppercase;
  user-select: none;
  position: relative;
}
.tile.filled {
  animation: zoom 0.2s;
}
.tile .front,
.tile .back {
  box-sizing: border-box;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}
.tile .front {
  border: 2px solid #d3d6da;
}
.tile.filled .front {
  border-color: #999;
}
.tile .back {
  transform: rotateX(180deg);
}
.tile.revealed .front {
  transform: rotateX(180deg);
}
.tile.revealed .back {
  transform: rotateX(0deg);
}

@keyframes zoom {
  0% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.shake {
  animation: shake 0.5s;
}

@keyframes shake {
  0% {
    transform: translate(1px);
  }
  10% {
    transform: translate(-2px);
  }
  20% {
    transform: translate(2px);
  }
  30% {
    transform: translate(-2px);
  }
  40% {
    transform: translate(2px);
  }
  50% {
    transform: translate(-2px);
  }
  60% {
    transform: translate(2px);
  }
  70% {
    transform: translate(-2px);
  }
  80% {
    transform: translate(2px);
  }
  90% {
    transform: translate(-2px);
  }
  100% {
    transform: translate(1px);
  }
}

.jump .tile .back {
  animation: jump 0.5s;
}

@keyframes jump {
  0% {
    transform: translateY(0px);
  }
  20% {
    transform: translateY(5px);
  }
  60% {
    transform: translateY(-25px);
  }
  90% {
    transform: translateY(3px);
  }
  100% {
    transform: translateY(0px);
  }
}

@media (max-height: 680px) {
  .tile {
    font-size: 3vh;
  }
}
</style>
