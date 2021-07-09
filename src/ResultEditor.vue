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
    <value-display :value="value" @mouse="onMouse" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
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

export default defineComponent({
  name: 'ValueEditor',
  components: { ValueDisplay },
  props: {
    title: String
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
