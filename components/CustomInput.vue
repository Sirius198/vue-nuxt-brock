<template>
  <ValidationProvider
    v-slot="{ errors, classes }"
    mode="eager"
    :rules="rules"
  >
    <span :symbol="symbol">
      <input
        :symbol="symbol"
        class="input"
        :value="value"
        :type="type"
        :placeholder="placeholder"
        :disabled="disabled"
        :readonly="readonly"
        :class="classes"
        @keyup.enter="$emit('event')"
        @input="setValue($event.target.value)"
      />
    </span>
    <span v-if="!doNotShowErrorMessage" class="error">{{ errors[0] }}</span>
  </ValidationProvider>
</template>

<script>
import { ValidationProvider } from 'vee-validate'
export default {
  name: 'CustomInput',
  components: { ValidationProvider },
  props: {
    value: {
      type: [String, Number],
      default: '',
    },
    type: {
      type: String,
      default: 'text',
    },
    placeholder: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    readonly: {
      type: Boolean,
      default: false,
    },
    rules: {
      type: [String, Object],
      default: '',
    },
    vid: {
      type: String,
      default: '',
    },
    doNotShowErrorMessage: {
      type: Boolean,
      default: false,
    },
    symbol: {
      type: String,
      default: '',
    },
  },
  methods: {
    setValue(inputValue) {
      this.$emit('input', inputValue)
    },
  },
}
</script>

<style lang="scss" scoped>
.input {
  width: 100%;
  padding: 10px 8px;
  font-size: $font-s;
  line-height: 20px;
  color: #000;
  border: 1px solid gainsboro;
  border-radius: 3px;
  outline: none;

  &:focus {
    border: 1px solid $firebrick;
  }

  &:disabled {
    background: transparent;
    cursor: not-allowed;
  }
}

.error {
  font-size: 12px;
  color: $firebrick;
}

.invalid {
  border: 1px solid $firebrick;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type='number'] {
  -moz-appearance: textfield;
}
</style>
