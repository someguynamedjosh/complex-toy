<template>
  <div class="value-editor-root">
    <div class="toolbar">
      <span class="title">{{
        ops.find((o) => o.name === modelValue).label
      }}</span>
      <span class="radioGroup">
        <button
          v-for="op in ops"
          :key="op.name"
          :class="{ selected: modelValue === op.name }"
          @click="modelValue = op.name"
        >
          {{ op.label }}
        </button>
      </span>
    </div>
    <value-display :value="result" />
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import { Point, Value } from './Data'
import ValueDisplay from './ValueDisplay.vue'

type OpName =
  | 'addition'
  | 'subtraction'
  | 'multiplication'
  | 'division'
  | 'exponentiation'
  | 'twoexp'
  | 'addexp';

interface Op {
  name: OpName;
  label: string;
}

function dooWop (op: OpName, val1: Point, val2: Point): Point {
  if (op === 'addition') {
    return {
      x: val1.x + val2.x,
      y: val1.y + val2.y
    }
  } else if (op === 'subtraction') {
    return {
      x: val1.x - val2.x,
      y: val1.y - val2.y
    }
  } else if (op === 'multiplication') {
    return {
      x: val1.x * val2.x - val1.y * val2.y,
      y: val1.x * val2.y + val1.y * val2.x
    }
  } else if (op === 'division') {
    // https://mathworld.wolfram.com/ComplexDivision.html
    const denominator = val2.x * val2.x + val2.y * val2.y
    return {
      x: (val1.x * val2.x + val1.y * val2.y) / denominator,
      y: (val1.y * val2.x - val1.x * val2.y) / denominator
    }
  } else if (op === 'exponentiation') {
    const exponented = {
      x: Math.exp(val2.x) * Math.cos(val2.y),
      y: Math.exp(val2.x) * Math.sin(val2.y)
    }
    return dooWop('multiplication', val1, exponented)
  } else if (op === 'twoexp') {
    const product = dooWop('multiplication', val1, val2)
    return dooWop('exponentiation', { x: 1, y: 0 }, product)
  } else if (op === 'addexp') {
    const product = dooWop('addition', val1, val2)
    return dooWop('exponentiation', { x: 1, y: 0 }, product)
  } else {
    throw new Error(`Unexpected op: ${op}`)
  }
}

export default defineComponent({
  name: 'ValueEditor',
  components: { ValueDisplay },
  props: {
    a: {
      type: Object as PropType<Value>,
      required: true
    },
    b: {
      type: Object as PropType<Value>,
      required: true
    }
  },
  computed: {
    result (): Value {
      const aPoints = []
      if (this.a.type === 'single' || this.a.type === 'vector') {
        aPoints.push(this.a.value)
      } else {
        aPoints.push(...this.a.value)
      }
      const bPoints = []
      if (this.b.type === 'single' || this.b.type === 'vector') {
        bPoints.push(this.b.value)
      } else {
        bPoints.push(...this.b.value)
      }
      const resultPoints = []
      for (const a of aPoints) {
        for (const b of bPoints) {
          resultPoints.push(dooWop(this.modelValue, a, b))
        }
      }

      if (resultPoints.length === 1) {
        return {
          type: this.a.type as unknown as 'single' | 'vector',
          value: resultPoints[0]
        }
      } else {
        return {
          type: 'multiple',
          value: resultPoints
        }
      }
    }
  },
  data () {
    return {
      ops: [
        { name: 'addition', label: 'A+B' },
        { name: 'subtraction', label: 'A-B' },
        { name: 'multiplication', label: 'A×B' },
        { name: 'division', label: 'A÷B' },
        { name: 'exponentiation', label: 'A𝑒ᴮ' },
        { name: 'addexp', label: '𝑒ᴬ⁺ᴮ' },
        { name: 'twoexp', label: '𝑒ᴬᴮ' }
      ] as Array<Op>,
      modelValue: 'addition' as OpName
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
