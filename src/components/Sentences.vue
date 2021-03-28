<template>
  <div id="sentence-app" ref="app">
    <p class="instructions">Put the words in the 
      <span>
        correct
        <div class="popup">
          <p>Some tasks have more than one solution, but the one that will count as "correct" is the one that sounds most natural when a sentence is taken out of context like that.</p>
        </div>
      </span> 
      order:</p>
    <p class="eng-sentence">{{ word.engSentence }}</p>
    <div class="draggable-words" :style="{ marginLeft: - 160 * draggables.length / 2 + 'px' }">
      <div v-for="(draggable, index) in draggables" class="draggable" :class="{ correct: correctItem[index] }" :key="draggable.id">
        <p :id="'button'+index" :ref="'button'+index">{{ draggable }}</p>
      </div>
    </div>
    <div class="static-words" :style="{ marginLeft: - 160 * draggables.length / 2 + 'px' }">
      <div v-for="(draggableGhost) in draggables" class="draggable disabled" :key="draggableGhost.id">
        <p>{{ draggableGhost }}</p>
      </div>
    </div>
    <div class="answer-fields" :style="{ marginLeft: - 160 * draggables.length / 2 + 'px' }">
      <div v-for="(field, index) in solution" :id="'field'+index" class="answer-field" :key="field.id">
        <p></p>
      </div>
    </div>
    <div class="sentence-button">
      <p v-if="!levelSolved" @click="checkAnswers()">Check...</p>
      <p v-if="levelSolved" @click="finishGame()">Continue...</p>
    </div>
  </div>
</template>

<script>
  /* eslint-disable */
import { Draggable } from 'gsap/all'
import { TweenLite } from 'gsap'
import { gsap } from 'gsap'
import { bus } from '../main'

export default {
  name: 'sentences',
  props: {
    word: {
      type: Object
    }
  },
  data() {
    return {
      solution: [],
      fieldStatus: [],
      correctItem: [],
      levelSolved: false
    }
  },
  created() {
    this.solution = this.word.sloSentence.split(' ');
    this.pushValues();
  },
  mounted() {
    gsap.registerPlugin(Draggable);
    this.doDragAndDrop();
  },
  methods: {
    pushValues() {
      for (let i = 0; i < this.solution.length; i++) {
        this.fieldStatus.push({ takenBy: null, correctlyTaken: false });
        this.correctItem.push(false);
      }
    },
    doDragAndDrop() {
      let solutionLength = this.solution.length;
      let fieldStatus = this.fieldStatus;

      for (let i = 0; i < solutionLength; i++) {
        Draggable.create(`#button${i}`, {
          type:"x,y",
          bounds: this.$refs.app,
          onDragStart() {
            // remove the button index from any fields it might have previously been part of
            for (let j = 0; j < solutionLength; j++) {
              if (fieldStatus[j].takenBy === i && !fieldStatus[j].correctlyTaken) {
                fieldStatus[j].takenBy = null;
              }
            }
          },
          onDragEnd(e) {
            let buttonLanding = false;
            for (let j = 0; j < solutionLength; j++) {
              let field = document.getElementById(`field${j}`);
              let offsetDiff = 160 * i;
              
              if (this.hitTest(`#field${j}`, '50%') && !fieldStatus[j].correctlyTaken) {
                TweenLite.to(`#button${i}`, 0.3, { x:field.offsetLeft - offsetDiff, y:80 });
                buttonLanding = true;
                // if any button currently taking up that field, send it back
                if (fieldStatus[j].takenBy !== null) {
                  let oldButtonIndex = fieldStatus[j].takenBy;
                  TweenLite.to(`#button${oldButtonIndex}`, 0.5, { x:0, y:0 });
                }
                fieldStatus[j].takenBy = i;
              }
            }
            if (!buttonLanding) {
              TweenLite.to(`#button${i}`, 1, { x:0, y:0 });
            }
          }
        })    
      }
    },
    checkAnswers() {
      let correctAnswers = 0;
      for (let i = 0; i < this.fieldStatus.length; i++) {
        if (this.draggables[this.fieldStatus[i].takenBy] === this.solution[i]) {
          this.fieldStatus[i].correctlyTaken = true;
          this.correctItem[this.fieldStatus[i].takenBy] = true;
          correctAnswers++;
          this.$forceUpdate();
        }
        else if (this.fieldStatus[i].takenBy === null) {
          continue;
        }
        else {
          TweenLite.to(`#button${this.fieldStatus[i].takenBy}`, 1, { x:0, y:0 });
          this.fieldStatus[i].takenBy = null;
        }
      }
      if (correctAnswers === this.solution.length) {
        this.levelSolved = true;
      }
    },
    finishGame() {
      bus.$emit('sentencesFinished');
    }
  },
  computed: {
    draggables() {
      let sentence = this.word.sloSentence
      let shuffledSentence = sentence.split(' ').sort(() => Math.random() - 0.5);
      return shuffledSentence;
    }
  }
}
</script>

<style lang="scss">

@import '../assets/sass/styles.scss';

#sentence-app {
  height: 100vh;
  overflow: hidden;

  .instructions {
    margin-top: 50px;
    text-align: center;
    font-size: 18px;

    span {
      background-image: linear-gradient(to right, black 33%, rgba(255,255,255,0) 0%);
      background-position: bottom;
      background-size: 3px 1px;
      background-repeat: repeat-x;
      cursor: pointer;

      .popup {
        display: none;
        position: absolute;
        top: 80px;
        left: 50%;
        margin-left: -150px;
        width: 300px;
        padding: 10px;
        background-color: #fff;
        z-index: 100;
      }

      &:hover > .popup { display: block; }
    }
  }

  .eng-sentence {
    margin-top: 10px;
    text-align: center;
    font-size: 25px;
    font-weight: 700;
  }

  .draggable-words, .static-words {
    position: absolute;
    top: 200px;
    left: 50%;

    .draggable {
      position: relative;
      display: inline-block;
      z-index: 1;

      p {
        width: 150px;
        padding-top: 10px;
        padding-bottom: 10px;
        margin-right: 10px;
        text-align: center;
        background-color: $blue;
        border-radius: 20px;
        box-sizing: border-box;
        border: 2px solid #fff;
        color: #fff;
        font-weight: 700;
      }

      &.correct {
        pointer-events: none;
        cursor: default;

        p {
          background-color: $gold;
          color: #000;
        }
      }

      &.disabled {
        z-index: 0;

        p {
          opacity: 0.4;
        }
      }
    }
  }

  .answer-fields {
    position: absolute;
    top: 280px;
    left: 50%;

    .answer-field {
      position: relative;
      display: inline-block;
      width: 150px;
      height: 46px;
      background-color: $grey;
      border-radius: 20px;
      opacity: 0.3;
      margin-right: 10px;
    }
  }

  .sentence-button {
    margin-top: 300px;
    cursor: pointer;

    p {
      text-align: center;
      color: #000;
      font-size: 25px;
      user-select: none;
      text-decoration: underline;

      &:hover {
        color: $grey;
      }
    }
  }
}

</style>
