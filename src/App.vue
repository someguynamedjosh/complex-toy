<template>
  <value-editor title="A" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import ValueEditor from './ValueEditor.vue'

export default defineComponent({
  name: 'App',
  components: { ValueEditor }
})
</script>

<style lang="scss">
body {
  padding: 0;
  margin: 0;
  overflow: hidden;
}
html, body, #app {
  width: 100%;
  height: 100%;
  background: black;
  color: white;
  font-family: sans;
}
button {
  border: none;
  border-radius: 0.5rem;
  background: #222;
  color: white;
  padding: 0.5rem 1rem;
  cursor: pointer;
  text-transform: uppercase;
}
button:hover {
  background: #444;
}

.toolbar {
  margin: 0.5rem;
  display: flex;
  column-gap: 0.5rem;
}

.radioGroup {
  display: flex;
}
.radioGroup > * {
  margin: 0;
  margin-right: -1px;
  border-radius: 0;
  background: transparent;
  border: 1px solid #222;
}
.radioGroup > :first-child {
  border-top-left-radius: 0.5rem;
  border-bottom-left-radius: 0.5rem;
}
.radioGroup > :last-child {
  border-top-right-radius: 0.5rem;
  border-bottom-right-radius: 0.5rem;
}
.radioGroup > .selected {
  background: #222;
}
.radioGroup > .selected:hover {
  background: #444;
}
</style>
