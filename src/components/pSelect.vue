<template>
  <div class="select_group">
    <div class="select_group_select" @click="model = !model">
      <div class="select_group_content">
        {{ select }}
      </div>
      <div class="select_group_expand" :class="{ show: model }"></div>
      <div
        class="select_group_popup"
        :class="{ show: model }"
        @click="model = !model"
      >
        <div class="select_group_popup_helper" @click="model = !model">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { ref } from "vue";
export default {
  setup() {
    return {
      model: ref(false),
    };
  },
  watch: {},
  props: {
    select: {
      type: String,
      required: false,
    },
    label: {
      type: String,
      required: false,
    },
  },
  methods: {},
  mounted() {},
};
</script>
<style lang="scss" scoped>
.select_group {
  &_content {
    width: 100%;
    height: 100%;
  }
  &_select {
    cursor: pointer;
    min-height: 20px;
    min-width: 100px;
    width: auto;
    flex: 10000 1 0%;
    align-self: stretch;
    max-width: 250px;
    text-align: center;
    padding: 5px 10px;
    border-radius: 4px;
    border: 1px solid rgba(0, 0, 0, 0.158);
    position: relative;
  }
  &_expand {
    position: absolute;
    right: 10px;
    top: 20%;
    width: 20px;
    height: 20px;
    background-image: url("../assets/expand.svg");
    background-size: cover;
    transform: rotate(180deg);
    transition: 0.3s transform;
    &.show {
      transform: rotate(0deg);
    }
  }
  &_popup {
    left: 0;
    top: 100%;
    position: absolute;
    max-height: 0;
    width: 100%;
    overflow-y: scroll;
    transition: 0.3s ease-in-out max-height;
    background: #fff;
    z-index: 1;
    border-radius: 4px;
    box-shadow: 0 1px 5px #0003, 0 2px 2px #00000024, 0 3px 1px -2px #0000001f;
    &.show {
      max-height: 200px;
    }
    &::-webkit-scrollbar {
      width: 6px;
      background: rgb(255, 255, 255);
    }
    &::-webkit-scrollbar-thumb {
      width: 6px;
      background: rgb(159, 157, 157);
      border-radius: 5px;
    }
  }
}
</style>
