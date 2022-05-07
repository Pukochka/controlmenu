<template>
  <div class="card" :style="{ background: color }">
    <slot></slot>
  </div>
</template>
<script>
  export default {
    setup() {
      return {};
    },
    props: {
      color: {
        type: String,
        required: false,
      },
    },
  };
</script>
<style lang="scss" scoped>
  .card {
    box-shadow: 0 1px 5px #0003, 0 2px 2px #00000024, 0 3px 1px -2px #0000001f;
    vertical-align: top;
    background: #fff;
    position: relative;
    border-radius: 4px;
    min-width: 40%;
  }
  @media (max-width: 400px) {
    .card.max {
      width: 100%;
    }
  }
</style>
