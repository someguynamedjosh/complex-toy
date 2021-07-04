<template>
  <vue-canvas @draw="draw" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import VueCanvas, { DrawEvent } from './VueCanvas.vue'

class GridDrawer {
  constructor (private event: CanvasRenderingContext2D, private divisionsPerWhole: number) {

  }
}

export default defineComponent({
  name: 'ValueDisplay',
  components: { VueCanvas },
  methods: {
    drawLine (event: DrawEvent, x1: number, y1: number, x2: number, y2: number) {
      const { ctx } = event
      ctx.beginPath()
      ctx.moveTo(x1, y1)
      ctx.lineTo(x2, y2)
      ctx.stroke()
    },
    draw (event: DrawEvent) {
      const { ctx, w, h } = event
      const cx = Math.floor(w / 2.0) + 0.5
      const cy = Math.floor(h / 2.0) + 0.5

      ctx.strokeStyle = '#fff'
      ctx.lineWidth = 1
      ctx.globalCompositeOperation = 'lighten'
      const smallLineColor = '#222'
      const mediumLineColor = '#555'
      const largeLineColor = '#fff'

      let [x, y] = [0, 0]
      let divisions = 0
      const divisionsPerWhole = 5
      const edge = 2.25
      const spacing = Math.min(w, h) / 2.0 / (edge * divisionsPerWhole)

      // Draw horizontal lines
      y = cy % spacing
      divisions = Math.floor((cy - y + 1.0) / spacing)
      while (y < h) {
        ctx.strokeStyle = divisions % divisionsPerWhole === 0 ? mediumLineColor : smallLineColor
        if (divisions === 0) ctx.strokeStyle = largeLineColor
        this.drawLine(event, 0, y, w, y)
        divisions -= 1
        y += spacing
      }
      // Draw vertical lines
      x = cx % spacing
      divisions = Math.floor((x - cx + 1.0) / spacing)
      while (x < w) {
        ctx.strokeStyle = divisions % divisionsPerWhole === 0 ? mediumLineColor : smallLineColor
        if (divisions === 0) ctx.strokeStyle = largeLineColor
        this.drawLine(event, x, 0, x, h)
        divisions += 1
        x += spacing
      }
    }
  }
})
</script>
