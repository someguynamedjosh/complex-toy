<template>
  <div id="the-main-part">
    <value-editor id="a" title="A" v-model="a" />
    <value-editor id="b" title="B" v-model="b" />
    <result-editor id="result" title="Result" :a="a" :b="b" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { Value } from './Data'
import ResultEditor from './ResultEditor.vue'
import ValueEditor from './ValueEditor.vue'

export default defineComponent({
  name: 'App',
  components: { ValueEditor, ResultEditor },
  data () {
    return {
      a: {
        type: 'single',
        value: { x: 0, y: 0 }
      } as Value,
      b: {
        type: 'single',
        value: { x: 0, y: 0 }
      } as Value
    }
  }
})
</script>

<style lang="scss" scoped>
#the-main-part {
  height: 100%;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr 1fr 1fr;
  grid-template-areas:
    "a"
    "b"
    "result";
}
@media (min-aspect-ratio: 0.8) {
  #the-main-part {
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 1fr;
    grid-template-areas:
      "a      b"
      "result result";
  }
}
@media (min-aspect-ratio: 2) {
  #the-main-part {
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 1fr;
    grid-template-areas: "a b result";
  }
}
#a {
  grid-area: a;
}
#b {
  grid-area: b;
}
#result {
  grid-area: result;
}
</style>
