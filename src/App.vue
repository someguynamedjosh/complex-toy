<template>
<value-display/>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import ValueDisplay from './ValueDisplay.vue'

export default defineComponent({
  name: 'App',
  components: { ValueDisplay }
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
}
</style>
