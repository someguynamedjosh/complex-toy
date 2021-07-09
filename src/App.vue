<template>
  <the-main-part />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import TheMainPart from './TheMainPart.vue'

export default defineComponent({
  name: 'App',
  components: { TheMainPart }
})
</script>

<style lang="scss">
body {
  padding: 0;
  margin: 0;
  overflow: hidden;
}
html, body, #app {
  width: 100vw;
  height: 100vh;
  background: black;
  color: white;
  font-family: sans;
  overflow: hidden;
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
