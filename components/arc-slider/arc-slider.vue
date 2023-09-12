<template>
  <canvas canvas-id="slider" :style="{ width: `${canvasSize}px`, height: `${canvasSize}px` }" @touchstart="onMouseDown"
    @touchmove="onMouseMove"></canvas>
</template>

<script>

/**
* @canvasSize canvas大小
* @r  大圆半径
* @dotR 滑块半径 
* @ratio 当前百分比
* @A 圆心坐标
* @lineWidth 线宽
* @startAngle 起始角度
* @allAngle 总角度
* @O 滑块起始点坐标
*/
let startAngle, allAngle, lineWidth, ctx, r, dotR, A, O;

export default {
  name: 'arc-slider',
  props: ['config'],
  data() {
    return {
      canvasSize: 0,
      ratio: 0
    }
  },
  methods: {
    init() {
      ctx = uni.createCanvasContext('slider', this)
      this.canvasSize = this.config.size / 750 * uni.getSystemInfoSync().windowWidth
      this.ratio = this.config.nowRatio
      startAngle = this.config.startAngle > 360 ? 360 : this.config.startAngle
      allAngle = this.config.allAngle > 360 ? 360 : this.config.allAngle
      r = this.canvasSize * .4
      dotR = r * this.config.dotRatio / 100
      lineWidth = r * this.config.lineRatio / 100
      A = { x: this.canvasSize / 2, y: this.canvasSize / 2 }
      O = { x: this.canvasSize / 2 + r, y: this.canvasSize / 2 }
      this.drawSlider()
    },
    drawSlider() {
      // 背景
      ctx.beginPath()
      ctx.setLineCap('round')
      ctx.setStrokeStyle(this.config.bgColor)
      ctx.setLineWidth(lineWidth)
      ctx.arc(A.x, A.y, r, startAngle / 180 * Math.PI, (allAngle + startAngle) / 180 * Math.PI)
      ctx.stroke()
      // 进度条
      ctx.beginPath()
      ctx.setStrokeStyle(this.config.actColor)
      ctx.arc(A.x, A.y, r, startAngle / 180 * Math.PI, ((this.ratio / 100) * (allAngle / 180) + (startAngle / 180)) * Math.PI)
      ctx.stroke()
      // 百分比
      ctx.font = `normal 500 ${r * .2 + 'px'} AlibabaPuHuiTi_2_65_Medium`
      ctx.setTextAlign('center')
      ctx.setTextBaseline('middle')
      ctx.fillText(`${this.ratio}%`, A.x, A.y)
      // 滑块
      ctx.beginPath()
      ctx.shadowBlur = 4
      ctx.shadowColor = "gray"
      ctx.setFillStyle(this.config.dotColor)
      ctx.arc(this.calcDotPosition().x, this.calcDotPosition().y, dotR, 0, 2 * Math.PI)
      ctx.fill()

      ctx.draw()
    },
    onMouseDown(e) {
      if (!this.config.canMove) return
      const { x, y } = e.touches[0]
      this.calcAngle(x, y)
    },
    onMouseMove(e) {
      if (!this.config.canMove) return
      const { x, y } = e.touches[0]
      this.calcAngle(x, y)
    },
    calcAngle(x, y) {
      const AB = Math.sqrt(Math.pow(A.x - O.x, 2) + Math.pow(A.y - O.y, 2)),
        AC = Math.sqrt(Math.pow(A.x - x, 2) + Math.pow(A.y - y, 2)),
        BC = Math.sqrt(Math.pow(O.x - x, 2) + Math.pow(O.y - y, 2)),
        cosA = (Math.pow(AB, 2) + Math.pow(AC, 2) - Math.pow(BC, 2)) / (2 * AB * AC)

      // 触摸的点到圆心的距离超过范围
      if (AC > r * 1.2 || AC < r * .7) return

      let angleA = Math.round(Math.acos(cosA) * 180 / (Math.PI))
      // 判断夹角在顺时针还是逆时针
      const dx = x - A.x;
      const dy = y - A.y;
      if ((O.x - A.x) * dy - (O.y - A.y) * dx < 0) {
        angleA = 360 - angleA
      }
      const tempAngle = angleA - startAngle < 0 ? Math.round((360 - startAngle + angleA) / allAngle * 100) : Math.round((angleA - startAngle) / allAngle * 100)
      if (tempAngle > 100) return
      this.ratio = tempAngle
      this.drawSlider()
    },
    calcDotPosition() {
      let cosA = Math.cos((startAngle + allAngle * this.ratio / 100) * 2 * Math.PI / 360),
        sinA = Math.sin((startAngle + allAngle * this.ratio / 100) * 2 * Math.PI / 360)
      return {
        x: (O.x - A.x) * cosA - (O.y - A.y) * sinA + A.x,
        y: (O.y - A.y) * cosA + (O.x - A.x) * sinA + A.y
      }
    }
  },
  beforeMount() {
    this.init()
  }
}
</script>

<style lang="scss"></style>