<template>
  <div id="hangman-app">
    <div class="hint">
      <span>{{ word.engWord }}</span>
    </div>
    <div id="paper">      
      <img class="paper-img" src="../assets/img/hangman/grey-tape.png">
      <div id="hangman-pics">
        <img :src="require(`../assets/img/hangman/${imgNumber}.png`)">
      </div>
      <div v-for="compLetter in compLetters" :key="compLetter.id" class="comp-letters">
        <span v-html="compLetter"></span>
      </div>
      <div id="endGame" v-show="displayMessage">
        <span>{{ gameEndMessage }}</span>
      </div>
    </div>

    <div id="user-console">
      <div class="user-letters" v-for="userLetter in userLetters.slice(0,8)" :key="userLetter.id" @click="userLetter.hex === 1 && guessLetter(userLetter.letter)">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`../assets/img/hangman/hex${userLetter.hex}.png`)">
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(8,17)" :key="userLetter.id" @click="userLetter.hex === 1 && guessLetter(userLetter.letter)">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`../assets/img/hangman/hex${userLetter.hex}.png`)">
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(17,25)" :key="userLetter.id" @click="userLetter.hex === 1 && guessLetter(userLetter.letter)">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`../assets/img/hangman/hex${userLetter.hex}.png`)">
      </div> 
    </div>

    <div class="continue-button">
      <p :class="{ disabled: !gameOver }" @click="goToNextTask()">Continue...</p>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { bus } from '../main'

export default {
  name: 'app',
  components: {

  },
  props: {
    word: {
      type: Object
    },
    hangmanSettings: {
      type: Array
    }
  },
  data () {
    return {
      compLetters: [],
      userLetters: [],
      allLetters: ["A", "B", "C", "Č", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "R", "S", "Š", "T", "U", "V", "Z", "Ž"],
      imgNumber: 0,
      displayMessage: false,
      gameEndMessage: '',
      currentWord: '',
      gameOver: false
    }
  },
  created() {
    this.generateUserArray();
    this.setup();
  },
  methods: {
    generateUserArray() {
      for (let i = 0; i < this.allLetters.length; i++) {
        this.userLetters.push({'letter': this.allLetters[i], 'hex': 1});
      }
    },
    setup() {
      this.currentWord = this.word.sloWord;
      for (let i = 0; i < this.currentWord.length; i++) {
        this.compLetters.push("&nbsp;");     
      }
      // if starting letters are on
      if (this.hangmanSettings[1]) {
        let randomLetter = Math.floor(Math.random() * this.currentWord.length);
        let letter = this.currentWord.charAt(randomLetter).toUpperCase();
        this.guessLetter(letter);
      }     
    },
    guessLetter(letter) {
      if (this.gameOver) return;

      // see which index a letter is at so its hexagon could be turned orange or grey
      let userIndex = this.allLetters.indexOf(letter);
      if (this.currentWord.toUpperCase().indexOf(letter) !== -1) {
        this.letterIsCorrect(userIndex, letter);
      } 
      else {
        this.letterIsWrong(userIndex, letter);
      }
    },
    letterIsCorrect(userIndex, letter) {
      // an array of all the indexes of a certain letter, because the letter can appear more than once within a word
      let indexes = [];
      // turn the hex orange
      this.userLetters[userIndex].hex = 2; 
      for (let i = 0; i < this.currentWord.length; i++) {
        if (this.currentWord.toUpperCase()[i] === letter) {
          indexes.push(i);
          for (let j = 0; j < indexes.length; j++) {
            let number = indexes[j];
            this.compLetters[number] = letter;
            this.checkIfVictory();
          }
        }
      }
    },
    letterIsWrong(userIndex, letter) {
      // turn the hex grey
      this.userLetters[userIndex].hex = 3;
      if (this.imgNumber === 5 && this.hangmanSettings[0] === 5) {
        this.imgNumber = 11;
        this.endGame('YOU LOST');          
      }
      else if (this.imgNumber < 10) {
        this.imgNumber++;
      }
      else {
        this.imgNumber = 11;
        this.endGame('YOU LOST');
      }
    },
    checkIfVictory() {
      if (this.compLetters.indexOf('&nbsp;') === -1) {
        this.endGame('VICTORY!');        
      }
    },
    endGame(message) {
      this.gameOver = true;
      this.displayMessage = true;
      this.gameEndMessage = message;
      for (let i = 0; i < this.currentWord.length; i++) {
        if (this.compLetters[i] === '&nbsp;') {
          this.compLetters[i] = this.currentWord[i].toUpperCase();
        }        
      }
    },
    goToNextTask() {
      if (!this.gameOver) return;

      if (this.gameEndMessage === 'VICTORY!') {
        bus.$emit('hangmanVictory', true);
      }
      else {
        bus.$emit('hangmanVictory', false);
      }
    }
  }
}
</script>

<style lang="scss">

@import '../assets/sass/styles.scss';

#hangman-app {
  border: 1px solid transparent;
  background-color: #CDDEEB;

  .hint {
    margin: 60px auto;
    text-align: center;

    span {
      font-size: 20px;
      font-weight: 700;
      color: #fff;
      background: $blue;
      border: 1px solid #fff;
      padding: 8px 16px;
    }
  }

  #paper {
    text-align: center;
    margin-top: -50px;
    height: 400px;

    .paper-img {
      position: relative;
    }

    #hangman-pics img {
      width: 120px;
      position: relative;
      top: -260px;
    }

    .comp-letters {
      font-family: 'Open Sans Condensed', sans-serif;
      position: relative;
      top: -220px;
      font-size: 1.8rem;
      border-bottom: 2px solid black;
      margin-right: 0.3rem;
      margin-left: 0.3rem;
      min-width: 1.3rem;
      min-height: 1rem;
      padding: 0 0.3rem;
      display: inline-block;
      text-align: center;
    }

    #endGame {
      font-family: 'Open Sans Condensed', sans-serif;
      font-size: 3rem;
      display: block;
      position: absolute;
      top: 250px;
      left: 50%;
      margin-left: -150px;
      transform: rotate(-20deg);
      width: 300px;
      -webkit-animation: fade-in 0.5s ease-in;
      -moz-animation: fade-in 0.5s ease-in;
      -o-animation: fade-in 0.5s ease-in;
      animation: fade-in 0.5s ease-in;   
      

      span {
        border: 4px solid #871205;
        color: #871205;
        line-height: 3rem;
        padding: 10px;
        padding-bottom: 0;
        background-color: #fff;
      }
    }
  }

  #user-console {
    display: block;
    text-align: center;
    margin-top: 20px;

    .user-letters {
      display: inline-block;
      font-size: 2rem;
      font-weight: bold;
      color: #fff;
      margin-top: -24px;
      width: 4.5rem;
      cursor: pointer;

      span {
        position: relative;
        display: inline-block;
        text-align: center;
        min-width: 3rem;
        top: 20px;
        z-index: 100;
        user-select: none;
      }

      .hexagon {
        position: relative;
        margin-top: -80px;
        user-select: none;
      }
    }
  }

  .continue-button {
    margin-top: 30px;
    margin-bottom: 30px;

    p {
      text-align: center;
      color: #000;
      font-size: 25px;
      user-select: none;
      cursor: pointer;

      &:hover {
        color: $grey;
      }

      &.disabled {
        opacity: 0.4;
        cursor: default;
        pointer-events: none;
      }
    }
  }
}

@keyframes cog-loop {
  0% {
    background-image: url("../assets/img/hangman/cog.png");
  }
  49% {
    background-image: url("../assets/img/hangman/cog.png");
  }
  50% {
    background-image: url("../assets/img/hangman/cog-turn.png");
  }
  99% {
    background-image: url("../assets/img/hangman/cog-turn.png");
  }
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@keyframes grow {
  0% {
    width: 0%;
    height: 0%;
  }
  100% {
    width: 400px;
    height: 430px;
  }
}

@keyframes appear {
  0% {
    opacity: 0;
  }
  75% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

</style>
