<template>
  <div class="input_danger" v-if="!model.required">
    {{
      model.value.length == model.max
        ? `Достигнуто максимальное количество символов`
        : `Минимальное количество символов ${model.min} введено ${model.value.length}`
    }}
  </div>
</template>
<script>
export default {
  setup() {
    return {};
  },
  props: {
    model: {
      type: Object,
      required: false,
    },
  },
};
</script>
<style lang="scss" scoped>
.input_danger {
  margin: 0;
  padding: 0;
  color: red;
  font-size: 11px;
}
</style>
