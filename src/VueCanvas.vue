<template>
  <div id="outer-box" ref="outerBox">
    <canvas ref="canvas" />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

export interface DrawEvent {
  ctx: CanvasRenderingContext2D,
  w: number,
  h: number
}

export default defineComponent({
  name: 'VueCanvas',
  emits: {
    /* eslint @typescript-eslint/no-unused-vars: 'off' */
    draw: (event: DrawEvent) => true
  },
  setup () {
    return {
      canvas: ref<HTMLCanvasElement | null>(null),
      outerBox: ref<HTMLElement | null>(null),
      context: null as CanvasRenderingContext2D | null,
      /* eslint @typescript-eslint/no-empty-function: 'off' */
      resizeListener: () => {},
      mounted: false
    }
  },
  mounted () {
    if (this.canvas) {
      this.updateCanvasSize()
      this.context = this.canvas.getContext('2d')
      this.resizeListener = () => this.updateCanvasSize()
      window.addEventListener('resize', this.resizeListener)
    }
    this.mounted = true
    requestAnimationFrame(() => this.draw())
  },
  unmounted () {
    this.mounted = false
    window.removeEventListener('resize', this.resizeListener)
  },
  methods: {
    updateCanvasSize () {
      if (this.canvas !== null && this.outerBox !== null) {
        this.canvas.width = this.outerBox.clientWidth
        this.canvas.height = this.outerBox.clientHeight
      }
    },
    draw () {
      // Eslint complains if I do this.data?.draw()
      if (this.context !== null) {
        const event = {
          ctx: this.context,
          w: this.context.canvas.width,
          h: this.context.canvas.height
        }
        event.ctx.globalCompositeOperation = 'source-over'
        event.ctx.fillStyle = '#000'
        event.ctx.fillRect(0, 0, event.w, event.h)
        this.$emit('draw', event)
      }
      if (this.mounted) {
        requestAnimationFrame(() => this.draw())
      }
    }
  }
})
</script>

<style lang="scss" scoped>
canvas {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}
#outer-box {
  position: relative;
}
</style>
