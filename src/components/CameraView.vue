<template>
  <div class="content">
    <div>
      <div>
        <div class="form-group">
          <div>
            <label for="inputImage" class="control-label">写真／動画</label>
          </div>
          <br>
          <div>
            <input type="file" class="form-control" id="inputImage" variant="success" accept="image/*, video/*" capture="camera" @change="onImageChanged">
          </div>
        </div>
        <br>
        <div>
          <button v-if="!preview" class='preview-button' v-on:click='previewMediaData'>プレビュー</button>
          <button v-if="preview" class='image-rotate' v-on:click='imageRotate'>回転</button>
          <br>
          <br>
          <div class="canvas-wrapper">
            <canvas id="previewCanvas" ref='previewCanvas' width=300px height=300px></canvas>
            <canvas id="drawCanvas" ref='drawCanvas' width=300px height=300px></canvas>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CameraView',
  data () {
    return {
      file: '',
      mediaData: null,
      image: null,
      imgHeight: 0,
      imgWidth: 0,
      previewCanvas: null,
      previewCtx: null,
      previewCanvasWidth: null,
      previewCanvasHeight: null,
      drawCanvas: null,
      drawCtx: null,
      drawCanvasWidth: null,
      drawCanvasHeight: null,
      preview: false,
      touchpositions: [],
      isDragging: false,
      // ドラッグ開始位置
      start: { x: 0, y: 0 },
      // ドラッグ中の位置
      diff: { x: 0, y: 0 },
      // ドラッグ終了後の位置
      end: { x: 0, y: 0 }
    }
  },
  methods: {
    onImageChanged (event) {
      console.log(event)
      if (event.target.files.length) {
        // 選択されたファイル情報を取得
        this.file = event.target.files[0]
        this.mediaData = new Image()
        let reader = new FileReader()
        reader.onload = (e) => {
          this.mediaData = e.target.result
          this.preview = false
        }
        reader.readAsDataURL(this.file)
      } else {
        console.log('no file selected')
      }
    },
    // redraw () {
    //   ctx.clearRect(0, 0, canvas.width, canvas.height);
    //   ctx.drawImage(img, diff.x, diff.y)
    // },
    _draw () {
      // this.previewCtx.drawImage(this.image, -this.imgWidth / 2, -this.imgHeight / 2, this.imgWidth, this.imgHeight)
      this.previewCtx.drawImage(this.image, -this.imgWidth / 2 + this.diff.x, -this.imgHeight / 2 + this.diff.y, this.imgWidth, this.imgHeight)
      this.drawCtx.clearRect(0, 0, this.drawCanvasWidth, this.drawCanvasHeight)
      this.drawCtx.strokeStyle = 'black'
      this.drawCtx.strokeRect(0, 0, this.drawCanvasWidth, this.drawCanvasHeight)
    },
    previewMediaData () {
      console.log('displayImage')
      this.previewCtx.clearRect(0, 0, this.previewCanvasWidth, this.previewCanvasHeight)
      this.image = new Image()
      this.image.onload = function () {
        let hRatio = this.previewCanvasWidth / this.image.width
        let vRatio = this.previewCanvasHeight / this.image.height
        let ratio = hRatio < vRatio ? hRatio : vRatio
        this.imgHeight = this.image.height * ratio
        this.imgWidth = this.image.width * ratio
        this.previewCtx.translate(this.previewCanvasWidth / 2, this.previewCanvasHeight / 2)
        this._draw()
      }.bind(this)
      this.image.src = this.mediaData
      this.preview = true
    },
    imageRotate () {
      console.log('imageRotate')
      this.previewCtx.clearRect(-this.imgWidth / 2 - 10, -this.imgHeight / 2 - 10, this.imgWidth + 20, this.imgHeight + 20)
      this.previewCtx.rotate(Math.PI / 2)
      this._draw()
    },
    getTouchPosition (touch) {
      let rect = this.drawCanvas.getBoundingClientRect()
      let x = touch.pageX - (rect.left + window.pageXOffset)
      let y = touch.pageY - (rect.top + window.pageYOffset)
      return {x: x, y: y}
    },
    touchStart (e) {
      console.log('touchStart')
      console.log(e)
      let touch = e.changedTouches[0]
      let pos = this.getTouchPosition(touch)
      this.touchpositions.push(pos)
      if (this.touchpositions.length === 2) {
        console.log(pos)
        console.log(this.drawCtx.getTransform())
        this.drawCtx.strokeStyle = 'red'
        let x = this.touchpositions[0].x < this.touchpositions[1].x ? this.touchpositions[0].x : this.touchpositions[1].x
        let y = this.touchpositions[0].y < this.touchpositions[1].y ? this.touchpositions[0].y : this.touchpositions[1].y
        let width = Math.abs(this.touchpositions[0].x - this.touchpositions[1].x)
        let height = Math.abs(this.touchpositions[0].y - this.touchpositions[1].y)
        this.drawCtx.strokeRect(x, y, width, height)
        this.touchpositions = []
      }
    },
    touchEnd (e) {
      console.log('touchEnd')
    },
    touchMove (e) {
      console.log('touchMove')
    },
    mouseDown (e) {
      console.log('mouseDown')
      this.isDragging = true
      this.start.x = e.clientX
      this.start.y = e.clientY
    },
    mouseMove (e) {
      console.log('mouseMove')
      if (this.isDragging) {
        this.diff.x = (e.clientX - this.start.x) + this.end.x
        this.diff.y = (e.clientY - this.start.y) + this.end.y
        this.previewCtx.clearRect(0, 0, this.previewCanvasWidth, this.previewCanvasHeight)
        this._draw()
      }
    },
    mouseUp (e) {
      console.log('mouseUp')
      this.isDragging = false
      this.end.x = this.diff.x
      this.end.y = this.diff.y
    }
  },
  mounted () {
    this.previewCanvas = this.$refs.previewCanvas
    this.previewCtx = this.previewCanvas.getContext('2d')
    this.previewCanvasWidth = this.$refs.previewCanvas.width
    this.previewCanvasHeight = this.$refs.previewCanvas.height
    this.drawCanvas = this.$refs.drawCanvas
    this.drawCtx = this.drawCanvas.getContext('2d')
    this.drawCanvasWidth = this.$refs.drawCanvas.width
    this.drawCanvasHeight = this.$refs.drawCanvas.height
    this.drawCtx.strokeStyle = 'black'
    this.drawCtx.strokeRect(0, 0, this.drawCanvasWidth, this.drawCanvasHeight)

    this.previewCanvas.addEventListener('mousedown', this.mouseDown, false)
    this.previewCanvas.addEventListener('mousemove', this.mouseMove, false)
    this.previewCanvas.addEventListener('mouseup', this.mouseUp, false)

    this.drawCanvas.addEventListener('touchstart', this.touchStart, {passive: true})
    this.drawCanvas.addEventListener('touchmove', this.touchMove, {passive: true})
    this.drawCanvas.addEventListener('touchend', this.touchEnd, false)
  }
}
</script>

<style scoped>
.content {
  text-align: left;
  padding: 0 0 0 1em;
  height: 100vh;
}
.form-control {
  vertical-align: top
}
.canvas-wrapper {
    position: relative;
}
.canvas-wrapper canvas {
    position: absolute;
    top: 0;
    left: 0;
}
</style>
