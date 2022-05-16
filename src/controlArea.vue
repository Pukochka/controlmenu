<template>
  <div class="body">
    <p-card class="card_area">
      <div class="flex justify-between">
        <div class="header">Настройки базового меню</div>

        <p-spinner :model="requestWaiting" />
      </div>

      <p-separator />
      <div class="messanger">
        <div class="messanger_message your">
          <div class="messanger_avatar">
            <div class="messanger_avatar_img"></div>
          </div>
          <div class="messanger_content your">
            <div class="messanger_content_text">Базовое меню</div>
            <div class="messanger_content_info">11.01 10:00</div>
          </div>
        </div>

        <div class="messanger_message">
          <div class="messanger_avatar">
            <div class="messanger_avatar_img"></div>
          </div>
          <div class="container">
            <div class="messanger_content">
              <div class="messanger_content_text">Меню</div>
              <div class="messanger_content_info">11.01 10:01</div>
            </div>
            <p-btn
              label="Настройки всего меню"
              color="#ddd"
              textcolor="#333"
              size="18px"
              margin="5px"
              padding="8px 0"
              @click="dialMenuSettings = !dialMenuSettings"
            >
            </p-btn>
            <div class="container_btn">
              <div
                class="container_btn_lines"
                v-for="(line, lineindex) in lines"
                :key="lineindex"
              >
                <div v-for="(btns, btnindex) in line" :key="btnindex">
                  <TransitionGroup
                    name="list"
                    tag="div"
                    class="container_btn_btns gutter"
                  >
                    <div
                      class="button delete"
                      title="Удалить линию"
                      v-if="line.length >= 2"
                      @click="
                        deleteLineDial = !deleteLineDial;
                        selectLine = btns.id;
                      "
                    >
                      <div class="button_helper"></div>
                      <span>+</span>
                    </div>

                    <div
                      class="button"
                      v-for="(btn, index) in btns.buttons"
                      :key="index"
                      :class="{ break: wordBreak(btn.route.message) }"
                      @click="selectBtn(btn, btns)"
                    >
                      <div class="button_helper"></div>
                      {{ btn.route.message }}
                    </div>
                    <div
                      v-if="btns.buttons.length < 10"
                      class="button"
                      @click="addBtn(btns.id)"
                    >
                      <div class="button_helper"></div>
                      Добавить кнопку
                    </div>
                  </TransitionGroup>
                </div>
              </div>
              <p-btn
                v-if="lines[0]?.length < 10"
                @click="addLine()"
                label="Добавить линию"
                margin="5px"
                size="18px"
                color="#ddd"
                textcolor="#333"
                padding="8px 0"
              />
              <p-btn
                padding="8px 0"
                label="Переместить кнопки"
                color="#ddd"
                size="18px"
                textcolor="#333"
                margin="5px"
                @click="dialDrag = !dialDrag"
              />
              <div class="danger" v-if="lines[0]?.length == 10">
                Достигнуто максимальное количество линий
              </div>
            </div>
          </div>
        </div>
      </div>
    </p-card>
  </div>

  <dial-menu-settings
    :model="dialMenuSettings"
    :settings="menuSettings"
    @closeMenuSettings="closeMenuSettings"
    @saveSettings="saveSettings"
  />

  <dial-drag
    :render="lines"
    :model="dialDrag"
    @closeDrag="closeDrag"
    @sortBtn="requestSortButton"
  />

  <dial-setting
    :lines="lines"
    :control="control"
    :model="dialSettings"
    :data="select"
    @closeSettings="closeSettings"
    @addLine="requestAddBtnWithLine"
    @addButton="requestAddBtnInLine"
    @updateButton="requestUpdateData"
    @deleteButton="requestDeleteButton"
  />
  <p-dialog :model="deleteLineDial">
    <p-card class="sure_card">
      <div class="sure">Вы уверены,что хотите удалить линию?</div>
      <div class="actions">
        <p-btn
          class="actions_btn"
          label="Отмена"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="deleteLineDial = !deleteLineDial"
        />
        <p-btn
          class="actions_btn"
          label="Подтвердить"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="
            requestDeleteLine(selectLine);
            deleteLineDial = !deleteLineDial;
          "
        />
      </div>
    </p-card>
  </p-dialog>
</template>
<script>
import axios from "axios";
import { ref } from "vue";
import dialSetting from "./components/dialSettings.vue";
import dialMenuSetting from "./components/dialMenuSettings.vue";
import dialDrag from "./components/dialDrag.vue";
import pBtn from "./components/pBtn.vue";
import pCard from "./components/pCard.vue";
import pDialog from "./components/pDialog.vue";
import pSeparator from "./components/pSeparator.vue";
import pSpinner from "./components/pSpinner.vue";

export default {
  components: {
    "dial-setting": dialSetting,
    "dial-menu-settings": dialMenuSetting,
    "dial-drag": dialDrag,
    "p-btn": pBtn,
    "p-card": pCard,
    "p-separator": pSeparator,
    "p-spinner": pSpinner,
    "p-dialog": pDialog,
  },
  setup() {
    return {
      lines: ref([]),
      dialSettings: ref(false),
      dialMenuSettings: ref(false),
      dialDrag: ref(false),
      deleteLineDial: ref(false),
      danger: ref(true),
      configButton: ref({}),
      selectLine: ref(0),
      requestWaiting: ref(false),
      menu: ref(0),
      menuSettings: ref({}),
      control: ref(false),
      select: ref({
        id: 0,
        line_id: 0,
        request_contact: false,
        request_location: false,
        request_poll: null,
        route: {
          id: 0,
          message: "",
          route: "",
        },
        sort: 0,
      }),
    };
  },
  props: {
    bot: Object,
    menu_id: Number,
    host: String,
    routes: Array,
  },
  methods: {
    closeMenuSettings() {
      this.dialMenuSettings = !this.dialMenuSettings;
    },
    closeSettings() {
      this.dialSettings = !this.dialSettings;
    },
    closeDrag() {
      this.dialDrag = !this.dialDrag;
    },
    saveSettings(input, size, time) {
      this.requestWaiting = true;
      try {
        axios
          .post(
            `https://api.bot-t.ru/v1/bot/keyboard/reply-keyboard/update?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA`,
            {
              bot_id: 889,
              menu_id: 26699,
              input_field_placeholder: input,
              resize_keyboard: size,
              one_time_keyboard: time,
            }
          )
          .then((response) => {
            this.lines = [];
            for (let line of JSON.parse(response.data).data) {
              this.lines.push(line.lines);

              this.menuSettings = line;
            }
            if (response.status == 200) {
              this.requestWaiting = false;
            }

            setTimeout(() => {
              this.scrollArea();
            }, 10);
          });
      } catch (err) {
        console.log(err);
      }
    },
    requestAddBtnInLine(line_id, message, route) {
      this.request(
        "add-button-in-line",
        null,
        line_id,
        null,
        message,
        route,
        null
      );
    },
    requestAddBtnWithLine(message, route) {
      this.request(
        "add-button-with-line",
        26699,
        null,
        null,
        message,
        route,
        null
      );
    },
    requestUpdateData(button_id, message, route) {
      this.request(
        "update-button",
        null,
        null,
        button_id,
        message,
        route,
        null
      );
    },
    requestSortButton(line, sort, button_id) {
      this.request(
        "change-sort-and-line",
        null,
        line,
        button_id,
        null,
        null,
        sort + ""
      );
    },
    requestDeleteButton(button_id) {
      this.request("delete-button", null, null, button_id, null, null, null);
    },
    requestDeleteLine(line_id) {
      this.danger = true;
      this.request("delete-line", null, line_id, null, null, null, null);
    },
    addBtn(line) {
      this.dialSettings = !this.dialSettings;
      this.control = true;
      this.select = {
        req: "add-button",
        line: line,
        request_contact: false,
        request_location: false,
        request_poll: null,
        route: {
          id: 0,
          message: "",
          route: "",
        },
      };
    },
    wordBreak(word) {
      if ([...word].includes(" ")) {
        return false;
      } else {
        return true;
      }
    },
    addLine() {
      this.dialSettings = !this.dialSettings;
      this.control = true;
      this.select = {
        req: "add-line",
        request_contact: false,
        request_location: false,
        request_poll: null,
        route: {
          message: "",
          route: "",
        },
      };
    },
    request(req, menu_id, line_id, button_id, message, route, sort) {
      this.requestWaiting = true;
      try {
        axios
          .post(
            `https://api.bot-t.ru/v1/bot/keyboard/reply-keyboard/${req}?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA`,
            this.createPostParams(
              menu_id,
              line_id,
              button_id,
              message,
              route,
              sort
            )
          )
          .then((response) => {
            this.lines = [];
            for (let line of JSON.parse(response.data).data) {
              this.lines.push(line.lines);
              this.menuSettings = line;
            }
            if (response.status == 200) {
              this.requestWaiting = false;
            }

            setTimeout(() => {
              this.scrollArea();
            }, 10);
          });
      } catch (err) {
        console.log(err);
      }
    },
    createPostParams(menu_id, line_id, id, message, route, sort) {
      let params = {
        bot_id: 889,
      };

      if (menu_id) {
        params.menu_id = menu_id;
      }
      if (line_id) {
        params.line_id = line_id;
      }
      if (id) {
        params.id = id;
      }
      if (message) {
        params.message = message;
      }
      if (route) {
        params.route = route;
      }
      if (sort) {
        params.sort = sort;
      }
      console.log(params);
      return params;
    },
    selectBtn(btn) {
      this.dialSettings = !this.dialSettings;
      this.select = btn;
      this.control = false;
    },
    scrollArea() {
      document.querySelectorAll(".container_btn_btns").forEach((ele) => {
        let pos = { left: 0, x: 0, y: 0 };

        const mouseDownHandler = function (e) {
          pos = {
            left: ele.scrollLeft,
            x: e.clientX,
          };
          document.addEventListener("mousemove", mouseMoveHandler);
          document.addEventListener("mouseup", mouseUpHandler);
        };

        const mouseMoveHandler = function (e) {
          const dx = e.clientX - pos.x;

          ele.scrollLeft = pos.left - dx;
        };

        const mouseUpHandler = function () {
          document.removeEventListener("mousemove", mouseMoveHandler);
          document.removeEventListener("mouseup", mouseUpHandler);
        };

        ele.addEventListener("mousedown", mouseDownHandler);
      });
    },
  },
  mounted() {
    this.request("view", 26699, null, null, null, null, null);
  },
};
</script>
<style lang="scss" scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.body {
  width: 100%;
  display: flex;
  justify-content: center;
  font-family: "Roboto", "sans-serif";
  direction: ltr;
  margin: 0;
  padding: 24px;
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
.header {
  padding: 8px 10px;
  font-size: 30px;
  font-weight: 500;
}
.card_area {
  width: 100%;
  max-width: 700px;
}
.danger {
  color: red;
  font-size: 16px;
}
.button {
  width: 100px;
  height: 40px;
  display: flex;
  margin: 4px;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: #ddd;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
  user-select: none;
  pointer-events: all;
  position: relative;
  word-break: break-word;
  &.break {
    word-break: break-all;
  }
  &:hover .button_helper {
    opacity: 1;
  }
  &_helper {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    border-radius: inherit;
    opacity: 0;
    background-color: rgba(0, 0, 0, 0.059);
    transition: background-color 0.3s cubic-bezier(0.25, 0.8, 0.5, 1),
      opacity 0.4s cubic-bezier(0.25, 0.8, 0.5, 1);
  }

  &.line {
    width: auto;
    flex-grow: 1;
    font-weight: 500;
  }
  &.delete {
    font-size: 24px;
    font-weight: 600;
    background: transparent;
    width: auto;
    height: auto;
    color: #333;
    padding: 5px;
    box-shadow: none;
    & span {
      transform: rotate(45deg);
    }
  }
  &.wh_auto {
    width: auto;
    height: auto;
  }
}
.messanger {
  &_message {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    padding: 10px;
    &.your {
      flex-direction: row-reverse;
    }
  }
  &_content {
    display: flex;
    justify-content: space-between;
    flex-direction: row;
    padding: 10px;
    border-radius: 20px;
    background-color: rgb(43, 156, 205);
    color: #fff;
    margin-left: 5px;
    &_info {
      padding-top: 10px;
      font-size: 16px;
      margin: 0 10px;
    }
    &.your {
      background-color: green;
      flex-direction: row-reverse;
      margin-right: 5px;
      margin-left: 0;
    }
    &_text {
      font-size: 22px;
    }
  }
  &_avatar {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    &_img {
      width: 40px;
      height: 40px;
      background-color: rgb(83, 81, 81);
      border-radius: 50%;
    }
  }
}
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}
.list-leave-active {
  position: absolute;
}
.container {
  flex-grow: 1;
  max-width: 96%;
  padding-right: 20px;
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
.container_btn {
  width: 100%;
  padding: 4px;
  margin-top: 5px;
  &_btns {
    display: -webkit-box;
    align-items: center;
    overflow-x: scroll;
    transition: 0.3s all;
    &::-webkit-scrollbar {
      height: 6px;
      background: rgb(255, 255, 255);
    }
    &::-webkit-scrollbar-thumb {
      height: 6px;
      background: rgb(169, 167, 167);
      border-radius: 5px;
    }
  }
}
@media (max-width: 700px) {
  .btnwid {
    max-width: 100px;
  }
}
@media (max-width: 440px) {
  .header {
    font-size: 26px;
  }
}
@media (max-width: 400px) {
  .header {
    font-size: 24px;
  }
  .text {
    font-size: 18px;
  }
  .info {
    font-size: 14px;
  }
}
</style>
<style></style>
