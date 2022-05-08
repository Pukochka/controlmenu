<template>
  <div class="body">
    <p-card class="card_area">
      <div class="flex justify-between">
        <div class="header">Настройки кнопок меню</div>

        <p-spinner :model="requestWaiting" />
      </div>

      <p-separator />
      <div class="messanger">
        <div class="messanger_message your">
          <div class="messanger_avatar">
            <div class="messanger_avatar_img"></div>
          </div>
          <div class="messanger_content your">
            <div class="messanger_content_text">Настройки меню</div>
            <div class="messanger_content_info">10.01 10:00</div>
          </div>
        </div>

        <div class="messanger_message">
          <div class="messanger_avatar">
            <div class="messanger_avatar_img"></div>
          </div>
          <div class="container">
            <div class="messanger_content">
              <div class="messanger_content_text">Меню</div>
              <div class="messanger_content_info">10.01 10:01</div>
            </div>
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
                      @click="selectBtn(btn, btns)"
                    >
                      <div class="button_helper"></div>
                      {{ btn.data.text }}
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
              <div
                class="button line"
                v-if="lines[0]?.length < 10"
                @click="addLine()"
              >
                <div class="button_helper"></div>
                Добавить линию
              </div>
              <div class="button line" @click="dialDrag = !dialDrag">
                <div class="button_helper"></div>
                Переместить кнопки
              </div>
              <div class="danger" v-if="lines[0]?.length == 10">
                Достигнуто максимальное количество линий
              </div>
            </div>
          </div>
        </div>
      </div>
    </p-card>
  </div>

  <dial-drag
    :render="lines"
    :model="dialDrag"
    @closeDrag="closeDrag"
    @sortBtn="requestSortButton"
  />

  <dial-setting
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
          padding="5px 10px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="deleteLineDial = !deleteLineDial"
        />
        <p-btn
          class="actions_btn"
          label="Подтвердить"
          padding="5px 10px"
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
  import axios from 'axios';
  import { ref } from 'vue';
  import dialSetting from './components/dialSettings.vue';
  import dialDrag from './components/dialDrag.vue';
  import pBtn from './components/pBtn.vue';
  import pCard from './components/pCard.vue';
  import pDialog from './components/pDialog.vue';
  import pSeparator from './components/pSeparator.vue';
  import pSpinner from './components/pSpinner.vue';

  export default {
    components: {
      'dial-setting': dialSetting,
      'dial-drag': dialDrag,
      'p-btn': pBtn,
      'p-card': pCard,
      'p-separator': pSeparator,
      'p-spinner': pSpinner,
      'p-dialog': pDialog,
    },
    setup() {
      return {
        lines: ref([]),
        dialSettings: ref(false),
        select: ref({
          data: {
            text: '',
            action: 'https://',
          },
          type: 0,
        }),
        control: ref(false),
        deleteLineDial: ref(false),
        danger: ref(true),
        configButton: ref({}),
        selectLine: ref(0),
        requestWaiting: ref(false),
        show: ref(false),
        drag: ref(false),
        dialDrag: ref(false),
        menu: ref(0),
      };
    },
    methods: {
      closeSettings() {
        this.dialSettings = !this.dialSettings;
      },
      closeDrag() {
        this.dialDrag = !this.dialDrag;
      },
      requestAddBtnInLine(line_id, type, text, action) {
        this.request(
          'add-button-in-line',
          null,
          line_id,
          null,
          type + '',
          text,
          action,
          null
        );
      },
      requestAddBtnWithLine( type, text, action) {
        console.log( this.menu)
        this.request(
          'add-button-with-line',
          this.menu,
          null,
          null,
          type + '',
          text,
          action,
          null
        );
      },
      requestUpdateData(button_id, text, action, type) {
        this.request(
          'update-data-and-type',
          null,
          null,
          button_id,
          type + '',
          text,
          action,
          null
        );
      },
      requestSortButton(line, sort, button_id) {
        this.request(
          'change-sort-and-line',
          null,
          line,
          button_id,
          null,
          null,
          null,
          sort + ''
        );
      },
      requestUpdateTypeButton(button_id, type) {
        this.request(
          'update-type-button',
          null,
          null,
          button_id,
          type,
          null,
          null,
          null
        );
      },
      requestDeleteButton(button_id) {
        this.request(
          'delete-button',
          null,
          null,
          button_id,
          null,
          null,
          null,
          null
        );
      },
      requestDeleteLine(line_id) {
        this.danger = true;
        this.request(
          'delete-line',
          null,
          line_id,
          null,
          null,
          null,
          null,
          null
        );
      },
      addBtn(line) {
        this.dialSettings = !this.dialSettings;
        this.control = true;
        this.select = {
          req: 'add-button',
          line: line,
          type: 0,
          data: {
            text: '',
            action: 'https://',
          },
        };
        console.log(line);
      },
      add(){
        const json = JSON.stringify();
        axios
          .post(
            'https://api.bot-t.ru/v1/bot/main/add-button-in-line?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA',{bot_id : 889 , line_id: 155623, type: 1,text:'dsadasd', action:'category/view?isNewMessage=true&id=100693' })
          .then((response) => {
            console.log(json)
            console.log(response)
            // for (let rout of JSON.parse(response.data).data) {
            //   console.log(rout);
            // }
            
          });
      },
      addLine() {
        this.dialSettings = !this.dialSettings;
        this.control = true;
        this.select = {
          req: 'add-line',
          type: 0,
          data: {
            text: '',
            action: 'https://',
          },
        };
      },
      request(req, menu_id, line_id, button_id, type, text, action, sort) {
        console.log(
          req +
            `bot_id=889${menu_id ? `&menu_id=${menu_id}` : ''}${
              line_id ? `&line_id=${line_id}` : ''
            }${button_id ? `&id=${button_id}` : ''}${
              type ? `&type=${type}` : ''
            }${text ? `&text=${text}` : ''}${
              action ? `&action=${action}` : ''
            }${sort ? `&sort=${sort}` : ''}`
        );
        this.requestWaiting = true;
        try {
          axios
            .post(
              `https://api.bot-t.ru/v1/bot/keyboard/inline-keyboard/${req}?token=1250754763:AAHCrhde6Hzz-PKOf-072dpZIFyPjh2obkA`,
              `bot_id=889${menu_id ? `&menu_id=${menu_id}` : ''}${
                line_id ? `&line_id=${line_id}` : ''
              }${button_id ? `&id=${button_id}` : ''}${
                type ? `&type=${type}` : ''
              }${text ? `&text=${text}` : ''}${
                action ? `&action=${action}` : ''
              }${sort ? `&sort=${sort}` : ''}`
            )
            .then((response) => {
              this.lines = [];
              for (let line of JSON.parse(response.data).data) {
                this.lines.push(line.lines);
                console.log(this.lines)
              }
              if (response.status == 200) {
                this.requestWaiting = false;
              }
              this.menu = JSON.parse(response.data).data[0].id

              setTimeout(() => {
                this.scrollArea();
              }, 10);
            });
        } catch (err) {
          console.log(err);
        }
      },
      selectBtn(btn) {
        this.dialSettings = !this.dialSettings;
        this.select = btn;
        this.control = false;
      },
      scrollArea() {
        document.querySelectorAll('.container_btn_btns').forEach((ele) => {
          let pos = { left: 0, x: 0, y: 0 };

          const mouseDownHandler = function (e) {
            pos = {
              left: ele.scrollLeft,
              x: e.clientX,
            };
            document.addEventListener('mousemove', mouseMoveHandler);
            document.addEventListener('mouseup', mouseUpHandler);
          };

          const mouseMoveHandler = function (e) {
            const dx = e.clientX - pos.x;

            ele.scrollLeft = pos.left - dx;
          };

          const mouseUpHandler = function () {
            document.removeEventListener('mousemove', mouseMoveHandler);
            document.removeEventListener('mouseup', mouseUpHandler);
          };

          ele.addEventListener('mousedown', mouseDownHandler);
        });
      },
    },
    mounted() {
      this.request('view', 577, null, null, null, null, null, null);
      this.add()
    },
  };
</script>
<style lang="scss" scoped>
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800&display=swap');
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  .body {
    width: 100%;
    display: flex;
    justify-content: center;
    font-family: 'Montserrat', 'sans-serif';
    direction: ltr;
  }
  body {
    position: relative;
    min-width: 150px;
    font-family: 'Montserrat', 'sans-serif';
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
    padding: 10px;
    font-size: 24px;
    font-weight: 500;
  }
  .bg-transparent {
    background: transparent !important;
  }
  .card_area {
    width: 100%;
    max-width: 700px;
  }
  .danger {
    color: red;
    font-size: 11px;
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
    font-size: 13px;
    border-radius: 4px;
    box-shadow: 0 1px 5px #0003, 0 2px 2px #00000024, 0 3px 1px -2px #0000001f;
    cursor: pointer;
    user-select: none;
    pointer-events: all;
    position: relative;
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
        font-size: 10px;
        margin: 0 10px;
      }
      &.your {
        background-color: green;
        flex-direction: row-reverse;
        margin-right: 5px;
        margin-left: 0;
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
    padding: 8px;
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
    background: #ccc;
    border-radius: 4px;
    box-shadow: 0 1px 5px #0003, 0 2px 2px #00000024, 0 3px 1px -2px #0000001f;
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
        background: rgb(159, 157, 157);
        border-radius: 5px;
      }
      &::-webkit-scrollbar-track {
        background: #ccc;
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
      font-size: 20px;
    }
  }
  @media (max-width: 400px) {
    .header {
      font-size: 18px;
    }
    .text {
      font-size: 13px;
    }
    .info {
      font-size: 9px;
    }
  }
</style>
<style></style>
