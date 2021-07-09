<template>
  <div class="value-editor-root">
    <div class="toolbar">
      <span class="title">{{ title }}</span>
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
  | 'power';

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
  } else {
    throw new Error('do it yourself')
  }
}

export default defineComponent({
  name: 'ValueEditor',
  components: { ValueDisplay },
  props: {
    title: String,
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
      if (this.a.type === 'single') {
        aPoints.push(this.a.value)
      } else {
        aPoints.push(...this.a.value)
      }
      const bPoints = []
      if (this.b.type === 'single') {
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
          type: 'single',
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
        { name: 'power', label: 'Aᴮ' }
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
