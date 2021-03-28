<template>
  <div id="settings" ref="settings">
    <div id="settings-all">
      <div class="close-wrapper" @click="close(false)">
        <span>X</span>
      </div>
      <div class="options">
        <p>Number of failed attempts:</p>
        <div v-on:click="numberOfAttempts(10)">
          <span>10</span>
          <img class="option-hex" :src="require(`../assets/img/hangman/hex${difficultyHex[0]}.png`)" />  
        </div>
        <div v-on:click="numberOfAttempts(5)">
          <span>5</span>
          <img class="option-hex" :src="require(`../assets/img/hangman/hex${difficultyHex[1]}.png`)" />
        </div>
      </div>
      <div class="options">
        <p>Start with one letter displayed:</p>
        <div @click="startingLetterOn(true)">
          <span>Y</span>
          <img class="option-hex" :src="require(`../assets/img/hangman/hex${startLetterHex[0]}.png`)" />
        </div>
        <div @click="startingLetterOn(false)">  
          <span>N</span>
          <img class="option-hex" :src="require(`../assets/img/hangman/hex${startLetterHex[1]}.png`)" />
        </div>
      </div>
      <div id="setting-button" @click="close(true)">Apply changes...</div>
      <div class="reset-button" @click="resetGame()">Reset game...</div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { bus } from '../main'

export default {
  name: 'settings',
  props: ['hangmanSettings'],
  data() {
    return {
      attempts: undefined,
      letterOn: undefined
    }
  },
  created() {
    this.attempts = this.hangmanSettings[0];
    this.letterOn = this.hangmanSettings[1];
  },
  methods: {
    numberOfAttempts(number) {
      this.attempts = number;
    },
    startingLetterOn(boolean) {
      this.letterOn = boolean;
    },
    close(save) {
      let settingsArray = save ? [this.attempts, this.letterOn] : this.hangmanSettings;
      bus.$emit('closeSettings', settingsArray);
    },
    resetGame() {
      if (confirm('This will delete all your progress. Continue?')) {
        bus.$emit('reset');
      }
    }
  },
  computed: {
    difficultyHex() {
      // 2 is a gold hex image (selected), and 3 is a grey one (unselected)
      return this.attempts === 10 ? [2, 3] : [3, 2];
    },
    startLetterHex() {
      return this.letterOn === true ? [2, 3] : [3, 2];
    }
  }
}
</script>

<style lang="scss">

@import '../assets/sass/styles.scss';

#settings {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  width: 400px;
  height: 450px;
  top: 50%;
  left: 50%;
  margin-top: -225px;
  margin-left: -200px;
  border: 4px solid #fff;
  box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);
  color: #fff;
  background-color: #686968;
  z-index: 500;
  text-align: center;

  .close-wrapper {
    font-family: 'Open Sans Condensed', sans-serif;
    position: absolute;
    right: 10px;
    top: 7px;
    width: 30px;
    font-weight: bold;
    font-size: 2rem;
    cursor: pointer;
    margin-bottom: 40px;

    &:hover {
      opacity: 0.5;
    }

    span {
      color: #fff;
    }
  }

  .options {
    text-align: center;
    margin-top: 30px;
    display: block;
    margin-bottom: -50px;
    font-size: 1.3rem;

    div {
      display: inline-block;
      text-align: center;
      width: 40px;
      cursor: pointer;
      height: 35px;
      margin: 15px 5px 0 5px;

      .option-hex {
        width: 40px;
        position: relative;
        top: -37px;
        user-select: none;
      }

      span {
        position: relative;
        z-index: 200;
        user-select: none;
      }
    }
  }

  #setting-button {
    margin-top: 60px;
    font-weight: bold;
    font-family: 'Open Sans Condensed', sans-serif;
    font-size: 20px;
    color: #fff;
    cursor: pointer;
    user-select: none;

    &:hover {
      opacity: 0.5;
    }
  }

  .reset-button {
    margin-top: 30px;
    font-family: 'Open Sans Condensed', sans-serif;
    font-size: 15px;
    color: #fff;
    cursor: pointer;
    user-select: none;

    &:hover {
      opacity: 0.5;
    }
  }
}

</style>
