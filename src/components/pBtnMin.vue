<template>
  <div class="close" :style="{ width: size, height: size, fontSize: font }">
    {{ icon }}
  </div>
</template>
<script>
export default {
  setup() {
    return {};
  },
  props: {
    icon: {
      type: String,
      required: false,
    },
    size: {
      type: String,
      required: false,
    },
    font: {
      type: String,
      required: false,
    },
  },
};
</script>
<style scoped>
.close {
  width: 20px;
  height: 20px;
  background-color: #fff;
  border-radius: 50%;
  transform: rotate(45deg);
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: 0.3s;
}
.close:hover {
  background-color: rgba(195, 188, 188, 0.399);
}
</style>
