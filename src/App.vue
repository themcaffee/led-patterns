<template>
  <div class="container" id="app">
    <h2 class="text-center">LED Pattern Generator</h2>
    <br>
    <div class="row">
      <form @submit.prevent="changeSize()" class="form-inline">
        <div class="col-5 form-group">
          <label class="col-5" for="height-input">Height</label>
          <input type="text" class="form-control col-5" id="height-input" v-model.number="heightInput" />
        </div>
        <div class="col-5 form-group">
          <label class="col-5" for="frames-input">Frames</label>
          <input type="text" class="form-control col-5" id="frames-input" v-model.number="framesInput" />
        </div>
        <button class="col-2 btn btn-default" type="submit">New template</button>
      </form>
    </div>
    <br>
    <div v-if="gridInitialized">
      <div class="row">
        <span class="col-3">
          <button class="btn btn-default" @click.prevent="addFrame()">Add Frame</button>
        </span>
        <span class="col-3">
          <button class="btn btn-default" @click.prevent="removeFrame()">Remove Frame</button>
        </span>
      </div>
      <br>
      <div class="row">
        <span class="col-3">
          <button class="btn btn-default" @click="randomize()">Randomize</button>
        </span>
        <span class="col-3">
          <button class="btn btn-default" @click="randomShift()">Random Shift</button>
        </span>
      </div>
      <br>
      <div class="row">
        <label class="col-2" for="color-input">Color</label>
        <input class="form-control col-3" type="text" v-model="colorInput" id="color-input" />
        <button class="btn btn-default col-2" @click.prevent="setBrushColor()">Set Color</button>
        <span class="color-box" :style="{ background: brushColor }">&nbsp;</span>
      </div>
      <br>
      <div v-for="row in leds.length" :key="row" class="row">
        <span v-for="frame in leds[0].length" :key="frame">
          <span class="led-box" :style="{ background: leds[row - 1][frame - 1] }" @click.prevent="setPixelColor(row - 1, frame - 1)">&nbsp;</span>
        </span>
      </div>
      <br>
      <h4>Output:</h4>
      <div class="row">
        {{ leds }}
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
      framesInput: 10,
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
      for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
        this.leds[rowIndex].push('#D3D3D3')
      }
    },
    removeFrame () {
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
