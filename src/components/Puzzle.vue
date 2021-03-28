<template>    
  <div id="puzzle-menu">
    <div class="close-wrapper" @click="close()">
      <span>X</span>
    </div>
    <div class="grid">
      <div v-for="(row, i) in grid" class="row" :key="row.id">
        <div v-for="(piece, j) in row" class="piece" :key="piece.id">
          <img :src="require(`../assets/img/puzzle/${i + j + 5 * i}.jpg`)" v-if="grid[i][j] === true" :class="{ fadeIn: wordIndex === i + j + 5 * i && word.strength === 1 && autoDisplay }">
        </div>
      </div>
    </div>
  </div>  
</template>

<script>
/* eslint-disable */
import dictionary from '../dictionary.json'
import { bus } from '../main'

export default {
  name: 'puzzle',
  props: {
    wordIndex: {
      type: Number
    },
    word: {
      type: Object
    },
    autoDisplay: {
      type: Boolean
    }
  },
  methods: {
    close() {
      bus.$emit('closePuzzle');
    }
  },
  computed: {
    grid() {
      // grid represents the puzzle layout - each subarray is a row, and each row has 6 items that are booleans - if the word corresponding to the puzzle piece has been learned, the boolean is set to true and so the puzzle piece is displayed
      let grid = [[], [], [], [], [], []];
      for (let i = 0; i < grid.length; i++) {
        for (let j = 0; j < 6; j++) {
          let boolean = dictionary.items[i + j + 5 * i].strength > 0 ? true : false;
          grid[i].push(boolean);
        }
      }
      return grid;
    }
  }
}
</script>

<style lang="scss">

@import '../assets/sass/styles.scss';

#puzzle-menu {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  width: 400px;
  height: 400px;
  top: 50%;
  left: 50%;
  margin-top: -200px;
  margin-left: -200px;
  border: 4px solid #fff;
  box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);
  background-color: $gold;
  z-index: 500;

  .close-wrapper {
    font-family: 'Open Sans Condensed', sans-serif;
    position: absolute;
    right: 0;
    top: 6px;
    width: 30px;
    font-weight: bold;
    font-size: 2rem;
    cursor: pointer;
    margin-bottom: 7px;

    &:hover {
      opacity: 0.5;
    }
  }

  .grid {
    width: 304px;
    border: 2px solid #fff;
    margin: auto;

    .row {
      margin: 0;
      height: 50px;

      .piece {
        position: relative;
        display: inline-block;
        width: 50px;
        height: 50px;
        background-color: rgba(255, 255, 255, 0.2);
        border: 1px solid rgba(0, 0, 0, 0.05);
        box-sizing: border-box;

        img {
          position: absolute;
          top: -1px;
          left: -1px;
          width: 50px;
          height: 50px;

          &.fadeIn {
            animation: fadeIn 2s linear;
          }
        }
      }
    }
  }
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

</style>
