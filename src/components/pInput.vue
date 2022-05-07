<template>
  <div class="label">{{ label }}</div>
  <input
    class="input"
    :style="{ borderBottom: `1px solid ${bordercolor};` }"
    :value="modelValue"
    :maxlength="+max"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
<script>
import { ref } from "vue";
export default {
  setup() {
    return {
      name: ref(""),
    };
  },
  mounted() {},
  watch: {},
  props: {
    max: {
      type: Number,
      required: false,
    },
    label: {
      type: String,
      required: false,
    },
    text: {
      type: String,
      required: false,
    },
    modelValue: {
      type: String,
      required: false,
    },
    color: {
      type: String,
      required: false,
    },
    textcolor: {
      type: String,
      required: false,
    },
    size: {
      type: String,
      required: false,
    },
    bordercolor: {
      type: String,
      required: false,
    },
  },
};
</script>
<style scoped>
.input {
  border: none;
  outline: none;
  height: 30px;
  width: 100%;
  font-size: 15px;
  border-bottom: 1px solid #aaa;
}
</style>
