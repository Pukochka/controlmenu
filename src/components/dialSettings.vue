<template>
  <p-dialog :model="model">
    <p-card>
      <div class="dial_header">
        {{ control ? "Добавление" : "Изменение" }} кнопки
      </div>
      <p-separator />
      <div class="dial_container">
        <p-input
          :label="(control ? 'Введите' : 'Измененить') + ' название кнопки'"
          v-model="message.value"
          :max="message.max"
        />
        <div class="danger" v-if="validateRepeatButton()">
          Кнопка с таким названием уже есть
        </div>
        <p-input-required :model="message" />
      </div>

      <div class="dial_container">
        <div class="dial_container_head">
          {{ (control ? "Добавить" : "Изменить") + " путь кнопки" }}
        </div>
        <p-select :select="selectRoute.text">
          <div
            class="actions_item"
            v-for="(item, index) in routes"
            :key="index"
            @click="selectRoutes(item)"
          >
            {{ item.text }}
          </div>
        </p-select>
        <div class="danger" v-if="dangerRoute">
          Для того чтобы {{ control ? "добавить" : "изменить" }} кнопку нужно
          выбрать действие
        </div>
      </div>

      <div class="dial_container" v-if="selectRoute.value?.length > 0">
        <div class="dial_container_head">
          {{ (control ? "Добавить" : "Изменить") + " путь кнопки" }}
        </div>

        <p-select :select="selectRouteText">
          <div
            class="actions_item"
            v-for="(item, key) in selectRoute.value[0]"
            :key="key"
            @click="selectRouteValue(item, key)"
          >
            {{ selectRoute.type == "category" ? item.text : item }}
          </div>
        </p-select>
        <div class="danger" v-if="dangerKey">
          Для того чтобы {{ control ? "добавить" : "изменить" }} кнопку нужно
          выбрать
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
          @click="closeSettings"
        />
        <p-btn
          class="actions_btn"
          v-if="!control"
          label="Удалить кнопку"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="sure = !sure"
        />
        <p-btn
          class="actions_btn"
          :label="control ? 'Добавить' : 'Изменить'"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="configButton(false)"
        />
      </div>
    </p-card>
  </p-dialog>
  <p-dialog :model="sure">
    <p-card class="sure_card">
      <div class="sure">
        Вы уверены,что хотите удалить кнопку {{ data?.route.message }}?
      </div>
      <div class="actions">
        <p-btn
          class="actions_btn"
          label="Отмена"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="sure = !sure"
        />
        <p-btn
          class="actions_btn"
          label="Подтвердить"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="
            configButton('delete');
            sure = !sure;
          "
        />
      </div>
    </p-card>
  </p-dialog>
</template>
<script>
import { ref } from "vue";
import axios from "axios";
import pBtn from "./pBtn.vue";
import pInput from "./pInput.vue";
import pSelect from "./pSelect.vue";
import pSeparator from "./pSeparator.vue";
import pDialog from "./pDialog.vue";
import pCard from "./pCard.vue";
import pInputRequired from "./pInputRequired.vue";

export default {
  components: {
    "p-btn": pBtn,
    "p-input": pInput,
    "p-select": pSelect,
    "p-separator": pSeparator,
    "p-dialog": pDialog,
    "p-card": pCard,
    "p-input-required": pInputRequired,
  },
  emits: [
    "changeType",
    "changeRoute",
    "closeSettings",
    "deleteButton",
    "addLine",
    "updateButton",
    "addButton",
  ],
  props: {
    model: {
      type: Boolean,
      required: false,
    },
    data: {
      type: Object,
      required: false,
    },
    lines: {
      type: Object,
      required: false,
    },
    control: {
      type: Boolean,
      required: false,
    },
  },
  setup() {
    return {
      btns: ref([]),
      dangerRoute: ref(false),
      dangerKey: ref(false),
      selectRouteKey: ref(""),
      selectRouteText: ref(""),
      routes: ref([]),
      sure: ref(false),
      currentRoute: ref(""),
      message: ref({
        value: "",
        required: false,
        min: 3,
        max: 20,
      }),
      selectRoute: ref({
        text: "Выберите путь",
        route: "",
        identifier: "",
        text_value: "",
        value: [],
      }),
    };
  },

  methods: {
    selectRouteValue(item, key) {
      this.dangerKey = false;
      if (this.selectRoute.type == "category") {
        this.selectRouteText = item.text;
        this.selectRouteKey = item.route;
      } else {
        this.selectRouteText = item;
        this.selectRouteKey = key;
      }
    },
    selectRoutes(route) {
      this.dangerRoute = false;
      if (!route.type) {
        this.selectRoute = route;
        this.selectRouteText = this.selectRoute.text_value
          ? this.selectRoute.text_value
          : "";
      } else if (route.type == "category") {
        this.selectRouteText = "Выбрать панель";
        this.selectRoute = {
          text: route.text,
          type: "category",
          value: [
            {
              ...route.value,
            },
          ],
        };
      }
    },
    closeSettings() {
      this.$emit("closeSettings");
    },
    getRoutes() {
      axios
        .post(
          "https://api.bot-t.ru/v1/bot/main/actions?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA",
          "bot_id=889&menu_id=26699"
        )
        .then((response) => {
          for (let rout of JSON.parse(response.data).data) {
            this.routes.push(rout);
          }
          console.log(this.routes);
        });
    },

    configButton(deletebtn) {
      this.dangerRoute = false;
      this.dangerKey = false;
      if (deletebtn) {
        this.$emit("deleteButton", this.data.id);
        this.closeSettings();
        return;
      }

      if (
        !this.validateDataRoute(this.selectRoute) ||
        this.validateRepeatButton() ||
        !this.message.required
      ) {
        return;
      }

      this.closeSettings();

      if (this.data.req == "add-line") {
        this.$emit(
          "addLine",
          this.message.value,
          this.chooseRoute(this.selectRoute)
        );
      } else if (this.data.req == "add-button") {
        this.$emit(
          "addButton",
          this.data.line,
          this.message.value,
          this.chooseRoute(this.selectRoute)
        );
      } else {
        this.$emit(
          "updateButton",
          this.data.id,
          this.message.value,
          this.chooseRoute(this.selectRoute)
        );
      }
    },
    chooseRoute(routes) {
      let route;
      if (routes.value?.length > 0) {
        if (routes.type) {
          route = this.selectRouteKey;
        } else {
          route = routes.route + routes.identifier + this.selectRouteKey;
        }
      } else {
        route = routes.route;
      }
      return route;
    },
    validateDataRoute(routes) {
      routes.route == ""
        ? (this.dangerRoute = true)
        : (this.dangerRoute = false);
      if (routes.value?.length > 0 && this.selectRouteKey == "") {
        this.dangerKey = true;
      }
      if (!this.dangerRoute && !this.dangerKey) {
        return true;
      } else {
        return false;
      }
    },
    validateRepeatButton() {
      let a = false;
      if (this.data.req) {
        for (let i = 0; i < this.btns.length; i++) {
          if (this.message.value == this.btns[i]) {
            a = true;
          }
        }
      } else {
        for (let btn of this.btns.filter((b) => b != this.data.route.message)) {
          if (this.message.value == btn) {
            a = true;
          }
        }
      }
      return a;
    },
    pushButtons() {
      this.btns = [];
      for (let btns of this.lines) {
        for (let btn of btns) {
          for (let item of btn.buttons) {
            this.btns.push(item.route.message);
          }
        }
      }
    },
  },
  watch: {
    model() {
      this.pushButtons();
      let categoryText = false;
      let category = true;

      this.message.value = this.data.route.message;
      this.selectRoute = this.routes.find(
        (r) => r.route == this.data.route.route
      );
      if (this.data.req == "add-button" || this.data.req == "add-line") {
        this.selectRoute = {
          text: "Выберите путь",
          route: "",
        };
      } else {
        if (this.selectRoute == undefined) {
          this.routes.forEach((r) => {
            if (r.value) {
              for (let item in r.value[0]) {
                if (r.route + r.identifier + item == this.data.route.route) {
                  categoryText = true;
                  category = false;
                  let delLength = item.length + r.identifier.length;
                  let delRoute = [...this.data.route.route];
                  delRoute.splice(-delLength, delLength);
                  this.currentRoute = delRoute.join("");
                  this.selectRouteText = r.value[0][item];
                }
              }
            }
          });
          this.selectRoute = this.routes.find(
            (r) => r.route == this.currentRoute
          );
          if (category) {
            this.routes.forEach((r) => {
              if (r.type) {
                categoryText = true;
                this.selectRoute = this.routes.find(
                  (rot) => rot.value == r.value
                );
                this.selectRoute = {
                  text: this.selectRoute.text,
                  type: "category",
                  value: [
                    {
                      ...this.selectRoute.value,
                    },
                  ],
                };

                for (let item in r.value) {
                  if (r.value[item].route == this.data.route.route) {
                    this.selectRouteText = r.value[item].text;
                  }
                }
              }
            });
          }
        }
        if (this.selectRoute.value?.length > 0 && !categoryText) {
          this.selectRouteText = this.selectRoute.text_value
            ? this.selectRoute.text_value
            : "";
        }
      }
    },
    "message.value"(value) {
      if (value.length < this.message.min) {
        this.message.required = false;
      } else if (value.length == this.message.max) {
        this.message.required = false;
      } else {
        this.message.required = true;
      }
    },
  },

  mounted() {
    this.getRoutes();
  },
};
</script>
<style lang="scss" scoped>
.dial {
  &_header {
    padding: 8px 10px;
    font-size: 24px;
    font-weight: 500;
  }
  &_container {
    padding: 10px;
    &_danger {
      color: red;
    }
    &_head {
      padding-bottom: 4px;
    }
  }
}
.area {
  position: relative;
  width: 95%;
  max-height: 150px;
}
.textarea {
  width: 100%;
  min-height: 50px;
  padding: 5px;
  resize: vertical;
  border-radius: 4px;
  outline: none;
  font-family: "Roboto";
  box-shadow: 0 1px 5px #0003, 0 2px 2px #00000024, 0 3px 1px -2px #0000001f;
  &_length {
    position: absolute;
    top: -17px;
    right: 0px;
    font-size: 12px;
  }
}
.center {
  display: flex;
  justify-content: center;
}
.sure {
  padding: 10px;
  &_card {
    min-width: 30%;
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
.danger {
  color: red;
  font-size: 11px;
}

.popup {
  &_item {
    padding: 5px 10px;
    width: 100%;
    font-size: 14px;
    transition: 0.3s background;
    cursor: pointer;
    background-color: #fff;
    position: relative;
    &:hover .popup_item_blur {
      opacity: 1;
    }
    &_blur {
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      background-color: rgba(74, 74, 74, 0.2);
      opacity: 0;
      transition: 0.3s opacity;
    }
  }
}
.full {
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.fullscreen {
  z-index: 6000;
  border-radius: 0 !important;
  max-width: 100vw;
  max-height: 100vh;
}
.keyboard-label {
  font-size: 18px;
}
@media (max-width: 440px) {
  .sure_card {
    font-size: 14px;
  }
  .actions_btn {
    font-size: 13px !important;
  }
}
</style>
