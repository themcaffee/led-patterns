<template>
  <div class="container" id="app">
    <h2 class="text-center">LED Pattern Generator</h2>
    <br>
    <div class="row">
      <form class="form-inline">
        <div class="col-5 form-group">
          <label class="col-5" for="height-input">Height</label>
          <input type="text" class="form-control col-5" id="height-input" v-model.number="height" />
        </div>
        <div class="col-5 form-group">
          <label class="col-5" for="frames-input">Frames</label>
          <input type="text" class="form-control col-5" id="frames-input" v-model.number="frames" />
        </div>
        <button class="col-2 btn btn-default" @click="changeSize()">Change Size</button>
      </form>
    </div>
    <div v-for="frame in leds" :key="frame" class="row">
      <div v-for="row in frame" :key="row">
        <span class="led-box">&nbsp;</span>
      </div>
    </div>
  </div>
</template>

<style>
.led-box {
  height: 25px;
  width: 25px;
  background-color: #D3D3D3;
  margin-left: 2px;
  margin-right: 2px;
  margin-top: 2px;
  margin-bottom: 2px;
  padding-left: 5px;
  padding-right: 5px;
  padding-bottom: 5px;
}
</style>

<script>
export default {
  name: 'App',
  data () {
    return {
      height: 49,
      frames: 1,
      leds: [[]]
    }
  },

  mounted () {
    this.changeSize()
  },

  methods: {
    changeSize () {
      this.leds = []
      for (var colIndex = 0; colIndex < this.frames; colIndex++) {
        let col = []
        for (var rowIndex = 0; rowIndex < this.height; rowIndex++) {
          col.push('D3D3D3')
        }
        this.leds.push(col)
      }
    }
  }
}
</script>
