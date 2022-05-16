<template>
  <p-dialog :model="model">
    <p-card>
      <div class="header">Настройки всего меню</div>
      <p-separator />
      <div class="menu_container">
        <div class="menu_container_item">
          <div class="">Изменение поля для ввода</div>
          <div class="">
            <p-input v-model="input.value" :max="input.max" />
          </div>
        </div>

        <div class="menu_container_item">
          <p-btn
            padding="10px 10px 10px 40px"
            label="Cкрывать меню после клика"
            color="transparent"
            textcolor="#222"
            @click="one_time = !one_time"
            ><div class="check" :class="{ checked: one_time }"></div>
          </p-btn>
        </div>
        <div class="menu_container_item">
          <p-btn
            padding="10px 10px 10px 40px"
            label="Изменение размера кнопокок"
            color="transparent"
            textcolor="#222"
            @click="size = !size"
            ><div class="check" :class="{ checked: size }"></div>
          </p-btn>
        </div>
      </div>
      <div class="actions">
        <p-btn
          class="actions_btn"
          label="Отмена"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="close"
        />
        <p-btn
          class="actions_btn"
          label="Сохранить"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="saveSettings"
        />
      </div>
    </p-card>
  </p-dialog>
</template>
<script>
import { ref } from "vue";
import pCard from "./pCard.vue";
import pDialog from "./pDialog.vue";
import pSeparator from "./pSeparator.vue";
import pBtn from "./pBtn.vue";
import pInput from "./pInput.vue";

export default {
  components: {
    "p-dialog": pDialog,
    "p-card": pCard,
    "p-btn": pBtn,
    "p-separator": pSeparator,
    "p-input": pInput,
  },

  props: {
    model: {
      type: Boolean,
      required: false,
    },
    settings: {
      type: Object,
      required: false,
    },
  },
  emits: ["closeMenuSettings", "saveSettings"],
  methods: {
    close() {
      this.$emit("closeMenuSettings");
    },
    saveSettings() {
      if (!this.input.required) {
        return;
      }
      this.close();
      this.$emit(
        "saveSettings",
        this.input.value == "" ? null : this.input.value,
        this.size,
        this.one_time
      );
    },
  },
  setup() {
    return {
      input: ref({
        value: "",
        required: false,
        min: 3,
        max: 64,
      }),
      size: ref(false),
      one_time: ref(false),
    };
  },
  watch: {
    model() {
      if (this.settings.input_field_placeholder) {
        this.input.value = this.settings.input_field_placeholder;
      } else {
        this.input.value = "";
      }

      this.size = this.settings.resize_keyboard;
      this.one_time = this.settings.one_time_keyboard;
    },
    "input.value"() {
      if (this.input.value.length == this.input.max) {
        this.input.required = false;
      } else {
        this.input.required = true;
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.header {
  font-weight: 500;
  padding: 8px 10px;
  font-size: 24px;
}
.wrapword {
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.flex {
  display: flex;
}
.justify {
  &-center {
    justify-content: center;
  }
  &-between {
    justify-content: space-between;
  }
}
.max {
  min-width: 60%;
}
.menu {
  &_container {
    // display: flex;
    // flex-direction: column;
    // align-items: center;
    padding: 10px;
  }
}
.actions {
  display: flex;
  justify-content: flex-end;
  align-self: end;
  padding: 4px;
  &_item {
    font-size: 14px;
    font-weight: 300;
    padding: 7px 0;
    transition: 0.3s background;
    &:hover {
      background: rgb(237, 237, 237);
    }
  }
}
.check {
  width: 14px;
  height: 14px;
  border: 2px solid #aaa;
  border-radius: 4px;
  position: absolute;
  top: 50%;
  left: 10px;
  transform: translateY(-50%);
  transition: 0.3s background-color;
  &.checked {
    border-color: #33af50;
    background-color: #80ed99;
  }
}
@media (max-width: 400px) {
  .header {
    font-size: 18px;
  }
}
</style>
