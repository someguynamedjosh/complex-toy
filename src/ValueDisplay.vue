<template>
  <vue-canvas
    @draw="draw"
    @mousedown="forwardMouseEvent"
    @mouseup="forwardMouseEvent"
    @mousemove="forwardMouseEvent"
    @wheel="onWheel"
  />
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import VueCanvas, { DrawEvent } from './VueCanvas.vue'
import { Point, Value } from './Data'

export interface GridMouseEvent {
  x: number;
  y: number;
  mouseDown: boolean;
}

export default defineComponent({
  name: 'ValueDisplay',
  components: { VueCanvas },
  props: {
    value: {
      type: Object as PropType<Value>,
      required: true
    }
  },
  data () {
    return {
      lastScale: 1.0,
      lastCenter: [1, 1],
      zoom: 1.0
    }
  },
  emits: {
    /* eslint @typescript-eslint/no-unused-vars: 'off' */
    mouse: (event: GridMouseEvent) => true
  },
  methods: {
    forwardMouseEvent (event: MouseEvent) {
      const x = (event.clientX - this.lastCenter[0]) * this.lastScale
      const y = -(event.clientY - this.lastCenter[1]) * this.lastScale
      const mouseDown = event.buttons > 0
      this.$emit('mouse', { x, y, mouseDown })
    },
    onWheel (event: WheelEvent) {
      this.zoom *= Math.exp(event.deltaY / -300.0)
    },
    drawLine (event: DrawEvent, x1: number, y1: number, x2: number, y2: number) {
      const { ctx } = event
      ctx.beginPath()
      ctx.moveTo(x1, y1)
      ctx.lineTo(x2, y2)
      ctx.stroke()
    },
    drawGrid (event: DrawEvent, scale: number) {
      const { ctx, w, h } = event
      // Compute some parameters.
      const cx = Math.floor(w / 2.0) + 0.5
      const cy = Math.floor(h / 2.0) + 0.5

      // Set up rendering mode
      ctx.lineWidth = 1
      ctx.globalCompositeOperation = 'lighten'
      ctx.fillStyle = '#fff'
      // Colors for different line weights
      const smallLineColor = '#222'
      const mediumLineColor = '#555'
      const largeLineColor = '#fff'

      // Values we will use in rendering the lines.
      let [x, y] = [0, 0]
      let divisions = 0
      const strongDivisionInterval = 5
      let unitsPerDivision = 0.5
      // Ew.
      if (this.zoom <= 1.0) unitsPerDivision = 1
      if (this.zoom <= 0.5) unitsPerDivision = 2
      if (this.zoom <= 0.2) unitsPerDivision = 5
      if (this.zoom <= 0.1) unitsPerDivision = 10
      if (this.zoom <= 0.05) unitsPerDivision = 20
      if (this.zoom <= 0.02) unitsPerDivision = 50
      if (this.zoom <= 0.01) unitsPerDivision = 100
      unitsPerDivision /= 5
      const spacing = 1.0 / (scale / unitsPerDivision)

      // Draw horizontal lines
      y = cy % spacing
      divisions = Math.floor((cy - y + 1.0) / spacing)
      while (y < h) {
        if (divisions % strongDivisionInterval === 0) {
          ctx.strokeStyle = largeLineColor
          this.drawLine(event, cx - spacing / 2, y, cx + spacing / 2, y)
          ctx.font = '1.5rem sans'
          const text =
            divisions === 0 ? '0' : `${unitsPerDivision * divisions}i`
          ctx.fillText(text, cx + 2.0, y - 5.0)
        }
        ctx.strokeStyle = smallLineColor
        if (divisions % (1.0 / unitsPerDivision) === 0) {
          ctx.strokeStyle = mediumLineColor
        }
        if (divisions === 0) ctx.strokeStyle = largeLineColor
        this.drawLine(event, 0, y, w, y)
        divisions -= 1
        y += spacing
      }
      // Draw vertical lines
      x = cx % spacing
      divisions = Math.floor((x - cx + 1.0) / spacing)
      while (x < w) {
        ctx.strokeStyle = smallLineColor
        if (divisions % strongDivisionInterval === 0) {
          ctx.strokeStyle = largeLineColor
          this.drawLine(event, x, cy - spacing / 2, x, cy + spacing / 2)
        }
        ctx.strokeStyle = smallLineColor
        if (divisions % (1.0 / unitsPerDivision) === 0) {
          ctx.strokeStyle = mediumLineColor
        }
        if (divisions === 0) ctx.strokeStyle = largeLineColor
        this.drawLine(event, x, 0, x, h)
        divisions += 1
        x += spacing
      }
    },
    translatePoint (
      event: DrawEvent,
      point: Point,
      scale: number
    ): { x: number; y: number } {
      const { ctx, w, h } = event
      const cx = Math.floor(w / 2.0) + 0.5
      const cy = Math.floor(h / 2.0) + 0.5
      return {
        x: cx + point.x / scale,
        y: cy - point.y / scale
      }
    },
    drawPoint (event: DrawEvent, point: Point, radius: number, scale: number) {
      const { ctx, w, h } = event
      const cx = Math.floor(w / 2.0) + 0.5
      const cy = Math.floor(h / 2.0) + 0.5
      const { x, y } = this.translatePoint(event, point, scale)

      ctx.beginPath()
      ctx.ellipse(x, y, radius, radius, 0.0, 0.0, Math.PI * 2.0)
      ctx.fill()
    },
    draw (event: DrawEvent) {
      const { ctx, w, h } = event
      const edge = 2.25 / this.zoom
      const scale = edge / (Math.min(w, h) / 2.0)
      this.lastScale = scale
      const bounds = ctx.canvas.getBoundingClientRect()
      this.lastCenter = [w / 2 + bounds.left, h / 2 + bounds.top]

      this.drawGrid(event, scale)

      ctx.fillStyle = '#ff0'
      ctx.globalCompositeOperation = 'source-over'
      if (this.value.type === 'single') {
        const radius = 0.1 / scale / this.zoom
        this.drawPoint(event, this.value.value, radius, scale)
      } else if (this.value.type === 'vector') {
        const { x: x0, y: y0 } = this.translatePoint(event, { x: 0, y: 0 }, scale)
        const { x, y } = this.translatePoint(event, this.value.value, scale)

        ctx.lineWidth = 0.3 / scale / this.zoom
        const angle = Math.atan2(y - y0, x - x0)
        ctx.strokeStyle = '#D400A4'
        ctx.beginPath()
        ctx.arc(x0, y0, 0.4 / scale / this.zoom, 0.0, angle, angle < 0.0)
        ctx.stroke()

        ctx.lineWidth = 5
        ctx.strokeStyle = '#0E76FF'
        ctx.fillStyle = ctx.strokeStyle
        ctx.beginPath()
        ctx.moveTo(x0, y0)
        ctx.lineTo(x, y)
        ctx.stroke()
        const radius = 0.1 / scale / this.zoom
        this.drawPoint(event, this.value.value, radius, scale)
      } else if (this.value.type === 'multiple') {
        const radius = 0.04 / scale / this.zoom
        let hue = 10.0
        for (const point of this.value.value) {
          ctx.fillStyle = `hsl(${hue}, 100%, 50%)`
          hue += 0.8
          this.drawPoint(event, point, radius, scale)
        }
      }
    }
  }
})
</script>
