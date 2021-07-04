<template>
  <div id="value-editor-root">
    <div class="toolbar">
      <button @click="clear">clear</button>
      <span class="radioGroup">
        <button
          :class="{selected: value.type === 'single'}"
          @click="makeSingle"
        >
          single
        </button>
        <button
          :class="{selected: value.type === 'multiple'}"
          @click="makeMultiple"
        >
          multiple
        </button>
      </span>
    </div>
    <value-display :value="value" @mouse="onMouse" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { Value } from './Data'
import ValueDisplay, { GridMouseEvent } from './ValueDisplay.vue'

export default defineComponent({
  name: 'ValueEditor',
  components: { ValueDisplay },
  data () {
    return {
      value: {
        type: 'single',
        value: { x: 0, y: 0 }
      } as Value
    }
  },
  methods: {
    onMouse (event: GridMouseEvent) {
      if (!event.mouseDown) return
      if (this.value.type === 'single') {
        const { x, y } = event
        this.value.value = { x, y }
      } else if (this.value.type === 'multiple') {
        const { x, y } = event
        this.value.value.push({ x, y })
      }
    },
    clear () {
      if (this.value.type === 'single') {
        this.makeSingle()
      } else if (this.value.type === 'multiple') {
        this.makeMultiple()
      }
    },
    makeSingle () {
      this.value = {
        type: 'single',
        value: { x: 0, y: 0 }
      }
    },
    makeMultiple () {
      this.value = {
        type: 'multiple',
        value: []
      }
    }
  }
})
</script>

<style lang="scss" scoped>
#value-editor-root {
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-rows: auto 1fr;
}
</style>