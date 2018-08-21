<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1> Image Pairing Game </h1>
    <h2>Game</h2>
    <div class="game-ip">
      <div class="game-ip__answers">
        <div class="game-ip__answers__item" v-for="image in images1" @click="select(image)">
          <img :src="require(`@/assets/img/${image.img}`)"
               :class="{ right: (image.pair !== null) && image.paired, wrong: (image.pair !== null) && !image.paired }"
               :style="{ cursor: (image.pair == null)?'pointer':'not-allowed' }"
               :ref="`image-1-${image.index}`">
        </div>
      </div>
      <hr>
      <div class="game-ip__answers">
        <div class="game-ip__answers__item" v-for="image in images2" @click="select(image)">
          <img :src="require(`@/assets/img/${image.img}`)"
               :class="{ right: (image.pair !== null) && image.paired, wrong: (image.pair !== null) && !image.paired }"
               :style="{ cursor: (image.pair == null)?'pointer':'not-allowed' }"
               :ref="`image-2-${image.index}`">
        </div>
      </div>
      <div id="svgContainer" ref="svgContainer">
        <svg ref="svg">
          <path v-for="line in paired" :ref="`path-${line.key}`" :class="{ right: line.paired, wrong: !line.paired }"/>
        </svg>
      </div>
    </div>
  </div>
</template>

<script>
import { cloneDeep } from 'lodash'

export default {
  name: 'app',
  data () {
    return {
      questions: [{
        answers: [
          {
            img1: 'eyes.jpeg',
            img2: 'glasses.jpg',
            paired: false,
            pair1: null,
            pair2: null,
          },
          {
            img1: 'feet.jpg',
            img2: 'shoes.jpeg',
            paired: false,
            pair1: null,
            pair2: null,
          },
          {
            img1: 'hands.jpg',
            img2: 'gloves.jpg',
            paired: false,
            pair1: null,
            pair2: null,
          },
          {
            img1: 'head.jpg',
            img2: 'hat.jpg',
            paired: false,
            pair1: null,
            pair2: null,
          }
        ],
        score: 0,
        answered: false,
        hit: false
      }],
      question: {},
      images1: [],
      images2: [],
      index: 0,
      ended: false,
      selected: null
    }
  },
  computed: {
    paired ({ _data: data  }) {
      const answers = cloneDeep(data.question.answers)
      if (answers) {
        const mapped = answers.map((a, key) => { return { key: key, ...a }})
        const filtered = mapped.filter(a => (a.pair1 !== null) || (a.pair2 !== null))
        return filtered
      }
      else return []
    }
  },
  methods: {
    setCurrent (questions) {
      let index = 0
      let qst = {}
      for (const key in questions) {
        if (!questions[key].answered) {
          qst = questions[key]
          break
        } else if (key == (questions.length - 1)) {
          qst = questions[key]
          this.ended = true
        }
      }
      this.$set(this, 'question', qst)
      this.splitAnswers(qst.answers)
    },
    splitAnswers (answers) {
      const copy = cloneDeep(answers)
      const arr1 = []
      const arr2 = []
      copy.map((a, index) => {
        arr1.push({ group: 1, index: index, img: a.img1, paired: a.paired, pair: a.pair1 })
        arr2.push({ group: 2, index: index, img: a.img2, paired: a.paired, pair: a.pair2 })
      })
      this.images1 = this.shuffle(arr1)
      this.images2 = this.shuffle(arr2)
    },
    shuffle (array) {
      for (let i = array.length-1; i >=0; i--) {
        var randomIndex = Math.floor(Math.random()*(i+1))
        var itemAtIndex = array[randomIndex]
        array[randomIndex] = array[i]
        array[i] = itemAtIndex
      }
      return array
    },
    select (img) {
      if (img.pair == null) {
        if (!this.selected) this.$set(this, 'selected', img)
        else {
          if (this.selected.group != img.group) {
            this.$set(img, 'pair', this.selected.index)
            this.$set(this.selected, 'pair', img.index)
            this.$set(this.question.answers[img.index], `pair${img.group}`, this.selected.index)
            this.$set(this.question.answers[this.selected.index], `pair${this.selected.group}`, img.index)
            if (img.index == this.selected.index) {
              this.$set(this.question, 'hit', true)
              this.$set(this.question.answers[img.index], 'paired', true)
              this.$set(img, 'paired', true)
              this.$set(this.selected, 'paired', true)
            } else {
              this.$set(this.question, 'hit', false)
              this.$set(this.question.answers[img.index], 'paired', false)
              this.$set(this.question.answers[this.selected.index], 'paired', false)
              this.$set(img, 'paired', false)
              this.$set(this.selected, 'paired', false)
            }
            this.$set(this, 'selected', null)
          }
        }
      }
    },
    signum (x) {
      return (x < 0) ? -1 : 1
    },
    absolute(x) {
      return (x < 0) ? -x : x
    },
    drawPath (svg, path, startX, startY, endX, endY) {
      var deltaX = (endX - startX) * 0.15
      var deltaY = (endY - startY) * 0.15
      // for further calculations which ever is the shortest distance
      var delta  =  deltaY < this.absolute(deltaX) ? deltaY : this.absolute(deltaX)

      // set sweep-flag (counter/clock-wise)
      // if start element is closer to the left edge,
      // draw the first arc counter-clockwise, and the second one clock-wise
      let arc1 = 0
      let arc2 = 1
      if (startX > endX) {
          arc1 = 1
          arc2 = 0
      }
      // draw tha pipe-like path
      // 1. move a bit down, 2. arch,  3. move a bit to the right, 4.arch, 5. move down to the end
      path.setAttribute("d",  "M"  + startX + " " + startY +
                              " V" + (parseFloat(startY) + parseFloat(delta)) +
                              " A" + delta + " " +  delta + " 0 0 " + arc1 + " " + (parseFloat(startX) + parseFloat(delta)*parseFloat(this.signum(deltaX))) + " " + (parseFloat(startY) + 2*parseFloat(delta)) +
                              " H" + (parseFloat(endX) - parseFloat(delta)*parseFloat(this.signum(deltaX))) +
                              " A" + delta + " " +  delta + " 0 0 " + arc2 + " " + endX + " " + (parseFloat(startY) + 3*parseFloat(delta)) +
                              " V" + endY)
    },
    connectElements (svg, path, startElem, endElem) {
      let svgContainer= this.$refs['svgContainer']
      // if first element is lower than the second, swap!
      if(startElem.offsetTop > endElem.offsetTop){
        const temp = startElem
        startElem = endElem
        endElem = temp
      }

      // get (top, left) corner coordinates of the svg container
      const svgTop  = svgContainer.offsetTop
      const svgLeft = svgContainer.offsetLeft

      // calculate path's start (x,y)  coords
      // we want the x coordinate to visually result in the element's mid point
      const startX = parseFloat(startElem.offsetLeft) + 0.5*parseFloat(startElem.offsetWidth) - parseFloat(svgLeft)    // x = left offset + 0.5*width - svg's left offset
      const startY = parseFloat(startElem.offsetTop)  + parseFloat(startElem.offsetHeight) - parseFloat(svgTop)     // y = top offset + height - svg's top offset

      // calculate path's end (x,y) coords
      const endX = endElem.offsetLeft + 0.5*endElem.offsetWidth - svgLeft
      const endY = endElem.offsetTop - svgTop

      // call function for drawing the path
      this.drawPath(svg, path, startX, startY, endX, endY)
    }
  },
  watch: {
    paired (paired) {
      setTimeout(() => {
        for (const key in paired) {
          const svg = this.$refs['svg']
          const path = this.$refs[`path-${paired[key].key}`][0]
          if (paired[key].pair1 !== null) {
            const indexStart = this.question.answers.findIndex(a => {
              return a.img1 == paired[key].img1
            })
            const elStart = this.$refs[`image-1-${indexStart}`][0]
            const elEnd = this.$refs[`image-2-${paired[key].pair1}`][0]
            this.connectElements(svg, path, elStart, elEnd)
          }
        }
      }, 50)
    }
  },
  mounted () {
    this.setCurrent(this.questions)
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

.game-ip {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  padding: 10px;
  hr {
    height: 10px;
    width: 100%;
    background-color: #ddd;
    border: none;
    border-radius: 15px;
    position: relative;
    z-index: -20;
  }
  &__answers {
    display: flex;
    &__item {
      flex-basis: calc(25% - 10px);
      flex-grow: 1;
      padding: 5px;
      img {
        width: calc(100% - 24px);
        height: calc(100% - 24px);
        margin: 10px;
        border-radius: 20px;
        border: 7px solid white;
        &.right {
          border-color: green;
        }
        &.wrong {
          border-color: red;
        }
      }
    }
  }
}

#svgContainer {
  height: 100%;
  width: 100%;
  z-index: -10;
  opacity:  0.75;
  margin: 2.5rem;
  position: absolute;
  top: 0;
  left: 0;
  svg {
    width: 100%;
    height: 100%;
    path {
      fill:   none;
      stroke-width: 7.5px;
      &.right {
        stroke: green;
      }
      &.wrong {
        stroke: red;
      }
    }
  }
}


</style>
