<template>
  <v-text-field label="قیمت قدیم" aria-label="Amount" required ref="inputRef" :rules="rules"/>
</template>

<script>
import { useCurrencyInput } from 'vue-currency-input'
export default {
  name: "CurrencyInput",
  props: {
    value: Number,
    options: Object,
    rules: Array,
  },
  setup(props) {
    const { inputRef } = useCurrencyInput(props.options)
    return { inputRef }
  }
}
</script>
