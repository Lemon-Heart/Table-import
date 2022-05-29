<template>
  <label class="inputFile">
    <input v-show="false" type="file" name="file" @change="onChange">
    <span>{{ label }}</span>
    <img :src="importIcon"/>
  </label>
</template>

<script>
import importIcon from '@/assets/svg/importIcon.svg'

export default {
  name: 'AppInputFile',
  props: {
    label: {
      type: String,
      default: 'Import File'
    }
  },
  setup(_, { emit }) {
    function onChange (event) {
      emit('onChange', event)
    }
    return { onChange, importIcon }
  }
}
</script>