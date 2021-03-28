<template>    
  <div id="stats" ref="stats">
    <div class="close-wrapper" @click="close()">
      <span>X</span>
    </div>
    <div class="arrow left" v-if="currentPage > 0" @click="changePage(-1)"></div>
    <div class="arrow left ghost"></div>
    <div v-for="(page, i) in dictPages" :key="page.id">
      <div class="page" v-if="currentPage === i">
        <div class="page-item" v-for="entry in page" :key="entry.id">
          <span class="slo-word" v-if="entry.strength > 0">{{ entry.sloWord }}</span>
          <span class="eng-word" v-if="entry.strength > 0">{{ entry.engWord }} </span>
          <img :src="require(`../assets/img/dictionary/${entry.strength}.png`)">
        </div>
      </div>
    </div>
    <div class="arrow right" v-if="currentPage < 2" @click="changePage(1)"></div>
    <div class="arrow right ghost"></div>
  </div>
</template>

<script>
/* eslint-disable */
import dictionary from '../dictionary.json'
import { bus } from '../main'

export default {
  name: 'sentences',
  data() {
    return {
      currentPage: 0
    }
  },
  methods: {
    changePage(value) {
      this.currentPage += value;
    },
    close() {
      bus.$emit('closeStats');
    }
  },
  computed: {
    dictPages() {
      let pages = [[], [], []];
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 12; j++) {
          pages[i].push(dictionary.items[i + j + 11 * i]);
        }
      }
      return pages;
    }
  }
}
</script>

<style lang="scss">

@import '../assets/sass/styles.scss';

#stats {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  width: 450px;
  height: 500px;
  top: 50%;
  left: 50%;
  margin-top: -250px;
  margin-left: -225px;
  border: 4px solid #fff;
  box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);
  background-color: $lightBlue;
  z-index: 500;

  .close-wrapper {
    font-family: 'Open Sans Condensed', sans-serif;
    position: absolute;
    right: 0;
    top: 7px;
    width: 30px;
    font-weight: bold;
    font-size: 2rem;
    cursor: pointer;
    color: $grey;

    &:hover {
      opacity: 0.8;
    }
  }

  .arrow {
    position: absolute;
    top: 235px;
    border-top: 15px solid transparent;
    border-bottom: 15px solid transparent;
    cursor: pointer;

    &.left {
      border-right: 15px solid $grey;
      left: 20px;

      &.ghost {
        z-index: -1;
        opacity: 0.4;
        cursor: default;
      }
    }

    &.right {
      border-left: 15px solid $grey;
      right: 20px;

      &.ghost {
        z-index: -1;
        opacity: 0.4;
        cursor: default;
      }
    }
  }

  .page {
    width: 340px;
    padding: 10px;
    background-color: #fff;

    .page-item {
      font-size: 16px;
      position: relative;
      height: 30px;
      padding-top: 5px;
      border-bottom: 1px solid $lightBlue;

      &:last-child {
        border-bottom: none;
      }

      .slo-word {
        position: absolute;
        margin-left: 30px;
      }

      .eng-word {
        position: absolute;
        margin-left: 167px;
      }

      img {
        float: right;
        padding-right: 20px;
      }
    }
  }
}
</style>
