<template>
  <div class="container" id="app">
    <br>
    <div class="row">
      <h2>LED Pattern Generator</h2>
    </div>
    <br>
    <div class="row">
      <form @submit.prevent="changeSize()" class="form-inline">
        <label class="col-1" for="height-input">Height</label>
        <input type="text" class="form-control col-2" id="height-input" v-model.number="heightInput" />

        <label class="offset-1 col-1" for="frames-input">Frames</label>
        <div class="input-group col-3">
          <input type="text" class="form-control" id="frames-input" v-model.number="framesInput" />
          <div class="input-group-append">
            <button class="btn btn-outline-secondary" @click.prevent="addFrame()" type="button">+</button>
            <button class="btn btn-outline-secondary" @click.prevent="removeFrame()" type="button">-</button>
          </div>
        </div>
        <button class="offset-1 col-2 btn btn-default" type="submit">New template</button>
      </form>
    </div>
    <br>
    <div v-if="gridInitialized">
      <div class="row">
        <div class="input-group col-3">
          <input class="form-control " type="text" v-model="colorInput" id="color-input" />
          <div class="input-group-append">
            <button class="btn btn-default" @click.prevent="setBrushColor()" type="button">Set Color</button>
          </div>
        </div>
        <span class="color-box" :style="{ background: brushColor }">&nbsp;</span>

        <div class="offset-1 col-2">
          <select class="form-control" v-model="effectInput">
            <option value="randomize">Randomize</option>
            <option value="random-shift">Random Shift</option>
          </select>
        </div>

        <button class="btn btn-default" @click="applyEffect()">Apply Effect</button>
      </div>
      <br>
      <div v-for="row in leds.length" :key="row" class="row">
        <span v-for="frame in leds[0].length" :key="frame">
          <span class="led-box" :style="{ background: leds[row - 1][frame - 1] }" @click.prevent="setPixelColor(row - 1, frame - 1)">&nbsp;</span>
        </span>
      </div>
      <br>
      <div class="row">
        <h4>Output:</h4>
        <div class="row">
          {{ leds }}
        </div>
      </div>
    </div>
    <br>
    <br>
  </div>
</template>

<style>
.led-box {
  float: left;
  height: 12px;
  width: 12px;
  margin-bottom: 2px;
  margin-left: 2px;
}
.color-box {
  float: left;
  height: 40px;
  width: 40px;
  margin-left: 2px;
}
</style>

<script>
export default {
  name: 'App',
  data () {
    return {
      heightInput: 49,
      framesInput: 30,
      effectInput: 'randomize',
      height: 0,
      frames: 0,
      leds: [],
      created: false,
      colorInput: '#AAAAFF',
      brushColor: '#AAAAFF'
    }
  },

  computed: {
    gridInitialized () {
      return !(this.height === 0 || this.frames === 0)
    }
  },

  methods: {
    changeSize () {
      if (this.heightInput <= 0 || this.framesInput <= 0) {
        return
      }
      this.height = this.heightInput
      this.frames = this.framesInput
      this.leds = []
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        let row = []
        for (var colIndex = 0; colIndex < this.frames; colIndex++) {
          row.push('#D3D3D3')
        }
        this.leds.push(row)
      }
    },
    addFrame () {
      this.frames += 1
      this.framesInput += 1
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        this.leds[rowIndex].push('#D3D3D3')
      }
    },
    removeFrame () {
      this.frames -= 1
      this.framesInput -= 1
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        this.leds[rowIndex].pop()
      }
    },
    setPixelColor (row, frame) {
      this.$set(this.leds[row], frame, this.brushColor)
    },
    setBrushColor () {
      if (this.colorInput.length !== 7) {
        return
      }
      this.brushColor = this.colorInput
    },
    /*
      Get a random hexadecimal color
    */
    getRandomColor () {
      let letters = '0123456789ABCDEF'
      let color = ''
      for (var index = 0; index < 6; index++) {
        color += letters[Math.floor(Math.random() * 16)]
      }
      return '#' + color
    },
    applyEffect () {
      if (this.effectInput) {
        this.randomize()
      } else {
        this.randomShift()
      }
    },
    randomize () {
      this.leds = []
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        let row = []
        for (var colIndex = 0; colIndex < this.frames; colIndex++) {
          row.push(this.getRandomColor())
        }
        this.leds.push(row)
      }
    },
    randomShift () {
      let newLeds = []
      // Randomize the first frame
      for (var firstRowIndex = 0; firstRowIndex < this.height; firstRowIndex++) {
        let firstRow = []
        firstRow.push(this.getRandomColor())
        newLeds.push(firstRow)
      }
      console.log('1')
      // Set the next frame with shifted pixels
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        let row = []
        for (var colIndex = 1; colIndex < this.frames; colIndex++) {
          console.log(rowIndex + ' ' + colIndex)
          if (rowIndex === this.height.length) {
            row.push(this.getRandomColor())
          } else {
            let pastColor = this.leds[rowIndex + 1][colIndex - 1]
            row.push(pastColor)
          }
        }
        newLeds.push(row)
      }
      console.log('2')
      for (var setRowIndex = 0; setRowIndex < this.height; setRowIndex++) {
        console.log(setRowIndex)
        this.$set(this.leds, rowIndex, newLeds[setRowIndex])
      }
    }
  }
}
</script>
