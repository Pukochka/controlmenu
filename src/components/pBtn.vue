<template>
  <div
    class="btn"
    ref="button"
    :style="{
      padding: padding,
      background: color,
      color: textcolor,
      fontSize: size,
      margin: margin,
      borderRadius: rad,
    }"
  >
    <slot></slot>
    <div class="" :class="{ btn_content: word }">{{ label }}</div>
    <div class="blur"></div>
    <div class="title" ref="title" v-if="title">{{ title }}</div>
  </div>
</template>
<script>
export default {
  setup() {
    return {};
  },
  methods: {},
  mounted() {},
  props: {
    label: {
      type: String,
      required: false,
    },
    rad: {
      type: String,
      required: false,
    },
    word: {
      type: Boolean,
      required: false,
    },
    blur: {
      type: Boolean,
      required: false,
    },
    title: {
      type: String,
      required: false,
    },
    padding: {
      type: String,
      required: false,
    },
    margin: {
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
  },
};
</script>
<style lang="scss" scoped>
.btn {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 5px;
  margin: 2px;
  color: #fff;
  border-radius: 4px;
  font-weight: 500;
  background-color: #aaa;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  user-select: none;
  text-align: center;
  &_content {
    word-break: break-word;
    max-width: 100px;
  }
  &:hover .blur {
    opacity: 1;
  }
  &:hover .label {
    visibility: visible;
  }
}
.title {
  position: absolute;
  background: #000;
  font-size: 13px;
  border: solid 1px #444;
  visibility: hidden;
  transition: 1s ease-in visibility;
}
.blur {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  opacity: 0;
  background-color: rgba(136, 136, 136, 0.23);
  transition: 0.3s opacity;
}
</style>
