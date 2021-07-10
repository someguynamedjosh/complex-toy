<template>
  <div class="value-editor-root">
    <div class="toolbar">
      <span class="title">{{ title }}</span>
      <button @click="clear">clear</button>
      <span class="radioGroup">
        <button
          :class="{ selected: modelValue.type === 'single' }"
          @click="makeSingle"
        >
          dot
        </button>
        <button
          :class="{ selected: modelValue.type === 'vector' }"
          @click="makeVector"
        >
          arrow
        </button>
        <button
          :class="{ selected: modelValue.type === 'multiple' }"
          @click="makeMultiple"
        >
          multiple
        </button>
      </span>
      <select v-model="animation" title="Animation" v-if="modelValue.type !== 'multiple'">
        <option value="none">No Animation</option>
        <option value="spin">Spin</option>
        <option value="wiggle">Wiggle</option>
        <option value="wobble">Wobble</option>
        <option value="figure8">Figure 8</option>
        <option value="spin+wobble">Spin+Wobble</option>
        <option value="spin+figure8">Spin+Figure 8</option>
        <option value="wiggle+wobble">Wiggle+Wobble</option>
        <option value="wobble+figure8">Wobble+Figure 8</option>
      </select>
    </div>
    <value-display :value="modelValue" @mouse="onMouse" />
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import { Point, Value } from './Data'
import ValueDisplay, { GridMouseEvent } from './ValueDisplay.vue'

type Animation = string;

function animTimer (): number {
  return (Date.now() / 9000) % 1.0
}

function complexMul (a: Point, b: Point): Point {
  return {
    x: a.x * b.x - a.y * b.y,
    y: a.x * b.y + a.y * b.x
  }
}

function euler (phase: number, amplitude: number): Point {
  return {
    x: amplitude * Math.cos(phase),
    y: amplitude * Math.sin(phase)
  }
}

export default defineComponent({
  name: 'ValueEditor',
  components: { ValueDisplay },
  props: {
    title: String,
    modelValue: {
      type: Object as PropType<Value>,
      required: true
    }
  },
  emits: ['update:modelValue'],
  data () {
    return {
      animation: 'spin' as Animation,
      animate: false,
      lastTime: animTimer()
    }
  },
  mounted () {
    this.animate = true
    this.doAnimation()
  },
  unmounted () {
    this.animate = false
  },
  methods: {
    doAnimation () {
      let last = this.lastTime
      const now = animTimer()
      if (this.modelValue.type === 'multiple') {
        this.animation = 'none'
      } else {
        let point = this.modelValue.value
        const TAU = Math.PI * 2.0
        if (this.animation.includes('figure8')) {
          last = 0.0
          point = {
            x: Math.cos(now * TAU * 1.0),
            y: Math.sin(now * TAU * 2.0) / 2.0
          }
        }
        if (this.animation.includes('spin')) {
          const lastPhase = last * TAU
          const nowPhase = now * TAU
          point = complexMul(point, euler(nowPhase - lastPhase, 1.0))
        }
        if (this.animation.includes('wiggle')) {
          const speed = 5.0
          // if (this.animation.includes('wobble')) speed = 2.0
          const scale = 0.3
          const lastPhase = Math.sin(speed * last * TAU) * scale
          const nowPhase = Math.sin(speed * now * TAU) * scale
          point = complexMul(point, euler(nowPhase - lastPhase, 1.0))
        }
        if (this.animation.includes('wobble')) {
          let speed = 2.0
          if (this.animation.includes('spin')) speed = 5.0
          if (this.animation.includes('figure8')) speed = 6.0
          const scale = 0.3
          const lastAmplitude = Math.sin(speed * last * TAU) * scale + 1.0
          const nowAmplitude = Math.sin(speed * now * TAU) * scale + 1.0
          point = complexMul(point, euler(0.0, nowAmplitude / lastAmplitude))
        }
        this.$emit('update:modelValue', {
          ...this.modelValue,
          value: point
        })
      }
      this.lastTime = now
      if (this.animate) {
        requestAnimationFrame(() => this.doAnimation())
      }
    },
    onMouse (event: GridMouseEvent) {
      if (!event.mouseDown) return
      if (
        this.modelValue.type === 'single' ||
        this.modelValue.type === 'vector'
      ) {
        const { x, y } = event
        this.$emit('update:modelValue', {
          type: this.modelValue.type,
          value: { x, y }
        })
      } else if (this.modelValue.type === 'multiple') {
        const { x, y } = event
        const oldPoints = this.modelValue.value
        this.$emit('update:modelValue', {
          type: 'multiple',
          value: [...oldPoints, { x, y }]
        })
      }
    },
    clear () {
      if (this.modelValue.type === 'single') {
        this.makeSingle()
      } else if (this.modelValue.type === 'multiple') {
        this.makeMultiple()
      }
    },
    makeSingle () {
      this.$emit('update:modelValue', {
        type: 'single',
        value: { x: 1, y: 0 }
      })
    },
    makeVector () {
      this.$emit('update:modelValue', {
        type: 'vector',
        value: { x: 0, y: 1 }
      })
    },
    makeMultiple () {
      this.$emit('update:modelValue', {
        type: 'multiple',
        value: []
      })
    }
  }
})
</script>

<style lang="scss" scoped>
.value-editor-root {
  border: 1px solid #888;
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-rows: auto 1fr;
}
.title {
  font-size: 2rem;
  font-weight: bold;
  margin: 0 1rem;
}
</style>
