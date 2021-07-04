<template>
  <vue-canvas
    @draw="draw"
    @mousedown="forwardMouseEvent"
    @mouseup="forwardMouseEvent"
    @mousemove="forwardMouseEvent"
  />
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import VueCanvas, { DrawEvent } from './VueCanvas.vue'
import { Point, Value } from './Data'

export interface GridMouseEvent {
  x: number,
  y: number,
  mouseDown: boolean,
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
      lastCenter: [1, 1]
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
      // Colors for different line weights
      const smallLineColor = '#222'
      const mediumLineColor = '#555'
      const largeLineColor = '#fff'

      // Values we will use in rendering the lines.
      let [x, y] = [0, 0]
      let divisions = 0
      const divisionsPerWhole = 5
      const spacing = 1.0 / (scale * divisionsPerWhole)

      // Draw horizontal lines
      y = cy % spacing
      divisions = Math.floor((cy - y + 1.0) / spacing)
      while (y < h) {
        ctx.strokeStyle = smallLineColor
        if (divisions % divisionsPerWhole === 0) {
          ctx.strokeStyle = largeLineColor
          this.drawLine(event, cx - spacing / 2, y, cx + spacing / 2, y)
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
        if (divisions % divisionsPerWhole === 0) {
          ctx.strokeStyle = largeLineColor
          this.drawLine(event, x, cy - spacing / 2, x, cy + spacing / 2)
          ctx.strokeStyle = mediumLineColor
        }
        if (divisions === 0) ctx.strokeStyle = largeLineColor
        this.drawLine(event, x, 0, x, h)
        divisions += 1
        x += spacing
      }
    },
    drawPoint (event: DrawEvent, point: Point, radius: number, scale: number) {
      const { ctx, w, h } = event
      const cx = Math.floor(w / 2.0) + 0.5
      const cy = Math.floor(h / 2.0) + 0.5

      ctx.beginPath()
      ctx.ellipse(
        cx + point.x / scale,
        cy - point.y / scale,
        radius,
        radius,
        0.0,
        0.0,
        Math.PI * 2.0
      )
      ctx.fill()
    },
    draw (event: DrawEvent) {
      const { ctx, w, h } = event
      const edge = 2.25
      const scale = edge / (Math.min(w, h) / 2.0)
      this.lastScale = scale
      const bounds = ctx.canvas.getBoundingClientRect()
      this.lastCenter = [w / 2 + bounds.left, h / 2 + bounds.top]

      this.drawGrid(event, scale)

      ctx.fillStyle = '#ff0'
      ctx.globalCompositeOperation = 'source-over'
      if (this.value.type === 'single') {
        const radius = 0.1 / scale
        this.drawPoint(event, this.value.value, radius, scale)
      } else if (this.value.type === 'multiple') {
        const radius = 0.04 / scale
        for (const point of this.value.value) {
          this.drawPoint(event, point, radius, scale)
        }
      }
    }
  }
})
</script>
