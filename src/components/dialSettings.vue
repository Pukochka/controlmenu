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
          v-model="text.value"
          :max="text.max"
        />
        <p-input-required :model="text" />
      </div>
      <div class="dial_container">
        <div class="dial_container_head">
          {{ (control ? "Добавить" : "Изменить") + " тип кнопки" }}
        </div>
        <p-select :select="selectAction.text">
          <div
            class="actions_item"
            v-for="(item, index) in actions"
            :key="index"
            @click="selectType(item)"
          >
            {{ item.text }}
          </div>
        </p-select>
        <div class="danger" v-if="dangerAction">
          Данный тип действия пока в разработке
        </div>
      </div>

      <div
        class="dial_container"
        v-if="selectAction.type == 2 || selectAction.type == 3"
      >
        <div class="dial_container_head">
          {{ (control ? "Добавить" : "Изменить") + " текст" }}
        </div>
        <div class="area">
          <textarea
            :maxlength="textarea.max"
            class="textarea"
            v-model="textarea.value"
          ></textarea>
          <div class="textarea_length">{{ textarea.outlength }}</div>
        </div>

        <p-input-required :model="textarea" />
      </div>

      <div class="dial_container" v-if="selectAction.type == 1">
        <div class="dial_container_head">
          {{ (control ? "Добавить" : "Изменить") + " готовый путь кнопки" }}
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
      <div class="dial_container" v-if="selectAction.type == 0">
        <p-input
          :label="(control ? 'Добавить' : 'Изменить') + ' ссылку кнопки'"
          v-model="action.value"
          :max="action.max"
        />
        <p-input-required :model="action" />
        <div class="danger" v-if="dangerActionValue">
          Не правильное значение ссылки пример : https://bot-t.ru
        </div>
      </div>
      <div
        class="dial_container"
        v-if="selectRoute.value?.length > 0 && selectAction.type == 1"
      >
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
        Вы уверены,что хотите удалить кнопку {{ data?.data.text }}?
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
    control: {
      type: Boolean,
      required: false,
    },
  },
  setup() {
    return {
      dangerRoute: ref(false),
      dangerKey: ref(false),
      dangerAction: ref(false),
      dangerActionValue: ref(false),
      selectRouteKey: ref(""),
      selectRouteText: ref(""),
      routes: ref([]),
      sure: ref(false),
      currentRoute: ref(""),
      type: ref(0),
      text: ref({
        value: "",
        required: false,
        min: 3,
        max: 20,
      }),
      action: ref({
        value: "",
        required: false,
        min: 8,
        max: 40,
      }),
      textarea: ref({
        value: "",
        required: false,
        min: 10,
        max: 100,
        outlength: 100,
      }),
      selectRoute: ref({
        text: "Выберите путь",
        route: "",
        identifier: "",
        text_value: "",
        value: [],
      }),
      selectAction: ref({
        text: "Ссылка",
        type: 0,
      }),
      actions: ref([
        {
          text: "Ссылка",
          type: 0,
        },
        {
          text: "Действие",
          type: 1,
        },
        {
          text: "Поделиться",
          type: 2,
        },
        {
          text: "Поделиться в группу",
          type: 3,
        },
        {
          text: "web",
          type: 4,
        },
      ]),
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
    selectType(action) {
      this.dangerAction = false;
      this.selectAction = action;
      if (this.selectAction.type > 3) {
        this.dangerAction = true;
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
      this.typeBtn = false;
      this.routeBtn = false;
      this.dangerAction = false;
    },
    getRoutes() {
      axios
        .post(
          "https://api.bot-t.ru/v1/bot/main/actions?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA",
          "bot_id=889&menu_id=577"
        )
        .then((response) => {
          for (let rout of JSON.parse(response.data).data) {
            this.routes.push(rout);
          }
          console.log(this.routes);
        });
    },

    configButton(deletebtn) {
      this.dangerActionValue = false;
      this.dangerRoute = false;
      this.dangerKey = false;
      if (deletebtn) {
        this.$emit("deleteButton", this.data.id);
        this.closeSettings();
        return;
      }
      if (this.selectAction.type > 3) {
        return;
      }
      if (!this.action.required || !this.text.required) {
        return;
      }
      if (this.selectAction.type == 2) {
        if (!this.textarea.required) {
          return;
        }
      }
      if (this.selectAction.type == 3) {
        if (!this.textarea.required) {
          return;
        }
      }
      if (!this.validateDataRoute(this.selectAction, this.selectRoute)) {
        return;
      }
      if (!this.validateAtionValue(this.action.value, this.selectAction.type)) {
        return;
      }

      this.closeSettings();

      if (this.data.req == "add-line") {
        this.$emit(
          "addLine",
          this.selectAction.type,
          this.text.value,
          this.chooseRoute(this.selectAction, this.selectRoute)
        );
      } else if (this.data.req == "add-button") {
        this.$emit(
          "addButton",
          this.data.line,
          this.selectAction.type,
          this.text.value,
          this.chooseRoute(this.selectAction, this.selectRoute)
        );
      } else {
        this.$emit(
          "updateButton",
          this.data.id,
          this.text.value,
          this.chooseRoute(this.selectAction, this.selectRoute),
          this.selectAction.type
        );
      }
    },
    chooseRoute(actions, routes) {
      let route;
      if (actions.type == 0) {
        route = this.action.value;
      } else if (actions.type == 1) {
        if (routes.value?.length > 0) {
          if (routes.type) {
            route = this.selectRouteKey;
          } else {
            route = routes.route + routes.identifier + this.selectRouteKey;
          }
        } else {
          route = routes.route;
        }
      } else if (actions.type == 2) {
        route = this.textarea.value;
      } else if (actions.type == 3) {
        route = this.textarea.value;
      } else if (actions.type == 4) {
        route = this.textarea.value;
      }
      return route;
    },
    validateDataRoute(actions, routes) {
      if (actions.type == 1) {
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
      } else {
        return true;
      }
    },
    validateAtionValue(value, type) {
      if (type == 0) {
        if (
          value.match(/(?:https?:\/\/)?(?:www\.)?[a-z0-9-]+\.(?:\.[a-z]{2,3})/)
        ) {
          return true;
        } else {
          this.dangerActionValue = true;
          return false;
        }
      } else {
        return true;
      }
    },
  },
  watch: {
    model() {
      let categoryText = false;
      let category = true;
      this.selectRoute = {
        text: "Выберите путь",
        route: "",
      };
      this.action.value = "https://";
      this.textarea.value = "Пример приветствия";
      this.selectAction = {};

      this.text.value = this.data.data.text;

      this.selectAction = this.actions.find(
        (act) => act.type == this.data.type
      );

      if (this.data.type == 0) {
        this.action.value = this.data.data.action;
      } else if (this.data.type == 1) {
        this.selectRoute = this.routes.find(
          (r) => r.route == this.data.data.action
        );
        if (this.selectRoute == undefined) {
          this.routes.forEach((r) => {
            if (r.value) {
              for (let item in r.value[0]) {
                if (r.route + r.identifier + item == this.data.data.action) {
                  categoryText = true;
                  category = false;
                  let delLength = item.length + r.identifier.length;
                  let delRoute = [...this.data.data.action];
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
                  if (r.value[item].route == this.data.data.action) {
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
      } else if (this.data.type == 2) {
        this.textarea.value = this.data.data.action;
      } else if (this.data.type == 3) {
        this.textarea.value = this.data.data.action;
      }
    },
    "text.value"(value) {
      if (value.length < this.text.min) {
        this.text.required = false;
      } else if (value.length == this.text.max) {
        this.text.required = false;
      } else {
        this.text.required = true;
      }
    },
    "action.value"(value) {
      if (value.length < this.action.min) {
        this.action.required = false;
      } else if (value.length == this.action.max) {
        this.action.required = false;
      } else {
        this.action.required = true;
      }
    },
    "textarea.value"(value) {
      this.textarea.outlength = this.textarea.max - value.length;
      if (value.length < this.textarea.min) {
        this.textarea.required = false;
      } else if (value.length == this.textarea.max) {
        this.textarea.required = false;
      } else {
        this.textarea.required = true;
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
@media (max-width: 440px) {
  .sure_card {
    font-size: 14px;
  }
  .actions_btn {
    font-size: 13px !important;
  }
}
</style>
