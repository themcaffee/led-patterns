<template>
  <div class="container" id="app" @mousedown.left="mouseDown()" @mouseup="mouseUp()">
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
    <div v-if="gridInitialized" class="row">
      <div class="form-inline">
        <label class="col-2">Pixel Height</label>
        <input type="text" class="form-control col-2" v-model.number="pixelHeight" />

        <label class="offset-1 col-2">Pixel Width</label>
        <input type="text" class="form-control col-2" v-model.number="pixelWidth" />
      </div>
    </div>
    <br>
    <div v-if="gridInitialized">
      <div class="row">
        <div class="col-2">
          <select class="form-control" v-model="effectInput">
            <option value="set-image">Image</option>
            <option value="randomize">Randomize</option>
            <option value="random-shift">Random Shift</option>
          </select>
        </div>

        <button class="col-2 btn btn-default" @click="applyEffect()">Apply Effect</button>

        <div class="col-5">
          <input :class="{ displayNone: effectInput !== 'set-image' }" type="file" @change="filesChange($event.target.files)" accept="image/*"/>
        </div>
      </div>
      <br>
      <div class="row">
        <sketch-picker v-model="colorInput" />
        <div class="offset-2 col-3" :class="{ displayNone: effectInput !== 'set-image' }">
          <img ref="imageElement" id="image-display" :src="imageUrl" :class="{ displayNone: imageUrl === '' }" @load="imageLoaded($event.target)"/>
          <canvas ref="imageCanvas" id="image-canvas"></canvas>
        </div>
      </div>
      <br>
      <div v-for="row in leds.length" :key="row" class="row">
        <span v-for="frame in leds[0].length" :key="frame">
          <span class="led-box" :style="{ background: leds[row - 1][frame - 1], width: pixelWidth + 'px', height: pixelHeight + 'px' }" @mouseover.prevent="setPixelColor(row - 1, frame - 1)"></span>
        </span>
      </div>
      <br>
      <div class="row">
        <button class="btn btn-default" @click="toggleShowOutput()">{{ showOutputText }}</button>
      </div>
      <div v-if="showOutput" class="row">
        <h4>Output:</h4>
        <textarea class="form-control" rows="10" v-model="leds"></textarea>
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
  -moz-user-select: -moz-none;
  -khtml-user-select: none;
  -webkit-user-select: none;

  /*
    Introduced in IE 10.
    See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
  */
  -ms-user-select: none;
  user-select: none;
}
#image-canvas {
  display: none;
}
.displayNone {
  display: none;
}
#image-display {
  height: 300px;
  width: 300px;
}
</style>

<script>
import { Sketch } from 'vue-color'

export default {
  name: 'App',
  data () {
    return {
      heightInput: 50,
      framesInput: 50,
      effectInput: 'set-image',
      fileInput: '',
      imageData: null,
      imageHexData: [],
      imageHexHeight: 0,
      imageHexWidth: 0,
      imageUrl: '',
      height: 0,
      frames: 0,
      leds: [],
      created: false,
      colorInput: {
        'hsl': {
          'h': 240,
          's': 1,
          'l': 0.5,
          'a': 1
        },
        'hex': '#0000FF',
        'rgba': {
          'r': 0,
          'g': 0,
          'b': 255,
          'a': 1
        },
        'hsv': {
          'h': 240,
          's': 1,
          'v': 1,
          'a': 1
        },
        'oldHue': 240,
        'source': 'hsva',
        'a': 1
      },
      isMouseDown: false,
      showOutput: false,
      pixelWidth: 12,
      pixelHeight: 12
    }
  },

  computed: {
    gridInitialized () {
      return !(this.height === 0 || this.frames === 0)
    },
    showOutputText () {
      if (this.showOutput) {
        return 'Hide Output'
      } else {
        return 'Show Output'
      }
    }
  },

  methods: {
    mouseDown () {
      this.isMouseDown = true
    },
    mouseUp () {
      this.isMouseDown = false
    },
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
      if (this.isMouseDown) {
        this.$set(this.leds[row], frame, this.colorInput.hex)
      }
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
      if (this.effectInput === 'randomize') {
        this.randomize()
      } else if (this.effectInput === 'random-shift') {
        this.randomShift()
      } else if (this.effectInput === 'set-image') {
        this.setImage()
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
    },
    filesChange (files) {
      // Setup reader to get base64 image data
      let reader = new FileReader()
      reader.onload = this.fileReaderLoaded
      reader.readAsDataURL(files[0])
    },
    fileReaderLoaded (event) {
      // Use base64 image data to set image element data src
      this.imageUrl = event.target.result
    },
    imageLoaded (img) {
      // Once image loaded get the byte information using an invisible canvas
      let imageCanvas = this.$refs['imageCanvas']
      imageCanvas.width = this.frames
      this.framesInput = this.frames
      imageCanvas.height = this.height
      this.heightInput = this.height
      this.imageHexHeight = imageCanvas.height
      this.imageHexWidth = imageCanvas.width

      let ctx = imageCanvas.getContext('2d')
      ctx.drawImage(img, 0, 0, imageCanvas.width, imageCanvas.height)

      let imageData = ctx.getImageData(0, 0, imageCanvas.width, imageCanvas.height)
      console.log(imageData)
      let data = imageData.data
      this.imageHexData = []
      for (var rowIndex = 0; rowIndex < this.imageHexHeight; rowIndex++) {
        let row = []
        for (var colIndex = 0; colIndex < this.imageHexWidth; colIndex++) {
          let currentPixel = this.getColorIndexForCoord(colIndex, rowIndex, this.imageHexWidth)
          let r = data[currentPixel[0]]
          let g = data[currentPixel[1]]
          let b = data[currentPixel[2]]
          let a = data[currentPixel[3]]
          let currentHex = this.rgbaToHex(r, g, b, a)
          row.push(currentHex)
        }
        this.imageHexData.push(row)
      }
    },
    componentToHex (c) {
      let hex = c.toString(16)
      return hex.length === 1 ? '0' + hex : hex
    },
    rgbaToHex (r, g, b, a) {
      return '#' + this.componentToHex(r) + this.componentToHex(g) + this.componentToHex(b) + this.componentToHex(a)
    },
    setImage () {
      this.leds = []
      for (var rowIndex = 0; rowIndex < this.imageHexHeight; rowIndex++) {
        let row = []
        for (var colIndex = 0; colIndex < this.imageHexWidth; colIndex++) {
          let currentPixel = this.imageHexData[rowIndex][colIndex]
          row.push(currentPixel)
        }
        this.leds.push(row)
      }
    },
    getColorIndexForCoord (x, y, width) {
      let red = y * (width * 4) + x * 4
      return [red, red + 1, red + 2, red + 3]
    },
    toggleShowOutput () {
      this.showOutput = !this.showOutput
    }
  },

  components: {
    'sketch-picker': Sketch
  }
}
</script>
