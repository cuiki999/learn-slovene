<template>
  <div id="app">
    <div v-if="part === 0">
      <div class="start-button" @click="startGame()">
        <span>LEARN A WORD</span>
      </div>
      <div class="menu">
        <img src="./assets/img/dictionary/bars.png" id="bars" @click="openMenu('stats')">
        <img src="./assets/img/hangman/cog.png" id="cog" @click="openMenu('settings')">
        <img src="./assets/img/puzzle/puzzle-piece2.png" id="puzzle" @click="openMenu('puzzle')">
      </div>
    </div>
    <hangman v-if="part === 1" :word="word" :hangmanSettings="hangmanSettings"></hangman>
    <sentences v-if="part === 2" :word="word"></sentences>
    <stats v-if="currentMenu === 'stats'"></stats>
    <settings v-if="currentMenu === 'settings'" :hangmanSettings="hangmanSettings"></settings>
    <puzzle v-if="currentMenu === 'puzzle'" :word="word" :wordIndex="wordIndex" :autoDisplay="autoDisplayPuzzle"></puzzle>
  </div>
</template>

<script>
  /* eslint-disable */
import dictionary from './dictionary.json'
import Hangman from './components/Hangman.vue'
import Sentences from './components/Sentences.vue'
import Stats from './components/Stats.vue'
import Settings from './components/Settings.vue'
import Puzzle from './components/Puzzle.vue'
import { bus } from './main'

export default {
  name: 'app',
  components: {
    'hangman': Hangman,
    'sentences': Sentences,
    'stats': Stats,
    'settings': Settings,
    'puzzle': Puzzle
  },
  data() {
    return {
      part: 0,
      items: dictionary.items,
      word: {},
      wordIndex: undefined,
      currentMenu: null,
      hangmanSettings: [10, false],
      puzzleComplete: false,
      autoDisplayPuzzle: false
    }
  },
  created() {
    bus.$on('closeSettings', (array) => {
      this.hangmanSettings = array;
      this.currentMenu = null;
      localStorage.setItem('hangmanSettings', JSON.stringify(array));
    });
    bus.$on('closeStats', () => this.currentMenu = null);
    bus.$on('closePuzzle', () => {
      this.currentMenu = null;
      this.autoDisplayPuzzle = false;
    });
    bus.$on('hangmanVictory', (boolean) => {
      this.part = boolean ? 2 : 0;
    });
    bus.$on('sentencesFinished', () => {
      this.part = 0;
      this.startAnew();
    });
    bus.$on('reset', () => this.resetGame());
  },
  mounted() {
    this.items.forEach(item => {
      item.strength = 0;
    });
    this.checkLocalStorage();
    this.easterEgg();
  },
  beforeDestroy() {
    bus.$off();
  },
  methods: {
    changeStrengths() {
      // read local storage to restore item strengths (how many times the word was learned)
      let tempDictionary = JSON.parse(localStorage.getItem('dictionary'));
      this.items.forEach((item, index) => {
        item.strength = tempDictionary.items[index].strength;
      });
    },
    getWord() {
      // avoid words that are at full strength (50 times), unless the puzzle has been solved, in which case avoid them only 10 times so we don't run the loop too many times
      let count = this.puzzleComplete ? 40 : 0;
      let random, word;

      while (count < 50) {
        random = Math.floor(Math.random() * this.items.length);
        word = this.items[random];
        if (word.strength < 3) {
          break;
        }
        count++;
      }

      this.wordIndex = random;
      return word;
    },
    async startGame() {
      this.word = await this.getWord();
      this.part = 1;
    },
    openMenu(menu) {
      this.currentMenu = menu;
    },
    checkLocalStorage() {
      if (localStorage.getItem('hangmanSettings') !== null) {
        this.hangmanSettings = JSON.parse(localStorage.getItem('hangmanSettings'));
      }
      if (localStorage.getItem('dictionary') !== null) {
        this.changeStrengths();
      }
    },
    startAnew() {
      let entry = this.items[this.wordIndex];
      if (entry.strength < 3) {
        this.items[this.wordIndex].strength++;
      }
      if (entry.strength === 1) {
        this.currentMenu = 'puzzle';
        this.autoDisplayPuzzle = true;
      }
      localStorage.setItem('dictionary', JSON.stringify(dictionary));
    },
    resetGame() {
      this.currentMenu = null;
      this.items.forEach(item => {
        item.strength = 0;
      });
      localStorage.clear();
    },
    easterEgg() {
      console.log("%c Good luck %c learning %c Slovene! ",  
      "background:white; border-left:1px solid black", 
      "background:blue; color:white;", 
      "background:red; color:white;" );
    }
  }
}
</script>

<style lang="scss">

@import 'assets/sass/styles.scss';

* {
  margin: 0;
}

#app {
  background-color: $lightBlue;
  height: 100vh;
  font-family: 'Open Sans Condensed', sans-serif;

  .start-button {
    position: absolute;
    top: 50%;
    left: 50%;
    margin-top: -25px;
    margin-left: -100px;
    width: 200px;
    height: 50px;
    text-align: center;
    background-color: $blue;
    border-radius: 20px;
    border: 2px solid #fff;
    cursor: pointer;

    &:hover {
      box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);
    }

    &:active {
      background-color: $gold;
    }

    span {
      font-size: 1.5rem;
      color: #fff;
      line-height: 55px;
      font-family: 'Neucha', sans-serif;
    }
  }

  .menu {
    position: absolute;
    width: 200px;
    height: 70px;
    left: 50%;
    top: 65%;
    margin-left: -100px;
    margin-top: -35px;


    #bars {
      left: 0;
    }

    #cog {
      left: 83px;
    }

    #puzzle {
      left: 170px;
    }

    #bars, #cog, #puzzle {
      padding-bottom: 15px;
      position: absolute;
      height: 35px;
      cursor: pointer;

      &:hover {

        border-bottom: 5px solid $blue;
      }
    }
  }
}
</style>
