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
          single
        </button>
        <button
          :class="{ selected: modelValue.type === 'multiple' }"
          @click="makeMultiple"
        >
          multiple
        </button>
      </span>
    </div>
    <value-display :value="modelValue" @mouse="onMouse" />
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import { Value } from './Data'
import ValueDisplay, { GridMouseEvent } from './ValueDisplay.vue'

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
  methods: {
    onMouse (event: GridMouseEvent) {
      if (!event.mouseDown) return
      if (this.modelValue.type === 'single') {
        const { x, y } = event
        this.$emit('update:modelValue', {
          type: 'single',
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
        value: { x: 0, y: 0 }
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
