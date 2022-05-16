<template>
  <p-dialog :model="model">
    <p-card class="max">
      <div class="flex justify-between">
        <div class="header">Перемещение кнопок</div>
        <p-btn
          class="actions_btn"
          padding="0"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="info = !info"
          ><div class="info" @click="info = !info"></div>
        </p-btn>
      </div>
      <p-separator />
      <div class="drag_container" v-for="(line, index) in render" :key="index">
        <div
          class=""
          v-for="(btns, btnindex) in line"
          :key="btnindex"
          @dragover="
            getLineId(btns);
            checkLength($event);
          "
          @touchend="
            getLineId(btns);
            checkLength($event);
          "
        >
          <draggable
            class="list-group"
            :list="btns.buttons"
            @change="log"
            group="people"
            itemKey="name"
          >
            <template #item="{ element }">
              <div class="button my-button">
                <div class="button_helper"></div>
                <div class="wrapword">{{ element.route.message }}</div>
              </div>
            </template>
          </draggable>
        </div>
      </div>
      <div class="actions">
        <p-btn
          class="actions_btn"
          label="Закрыть"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="close"
        />
      </div>
    </p-card>
  </p-dialog>
  <p-dialog :model="info">
    <p-card>
      <div class="header">Помощь по перемещению</div>
      <p-separator />
      <p class="info_content">
        Для того чтобы переместить кнопку, просто наведите на неё и зажмите
        левую кнопку мыши, затем переместите на нужное место.
      </p>
      <p class="info_content">
        Максимальное количество кнопок в стоке 10, если их будет больше,
        изменения не будут учитываться.
      </p>
      <p class="info_content">
        Если в строке не осталось кнопок, строка автоматически удаляется.
      </p>
      <p class="info_content">
        Рекомендуется использовать эту функцию на персональном компьютере.
      </p>
      <div class="actions">
        <p-btn
          class="actions_btn"
          label="Закрыть"
          padding="8px 16px"
          color="transparent"
          textcolor="#222"
          size="14px"
          @click="info = !info"
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
import draggable from "vuedraggable";
import pBtn from "./pBtn.vue";

export default {
  components: {
    "p-dialog": pDialog,
    "p-card": pCard,
    "p-btn": pBtn,
    "p-separator": pSeparator,
    draggable,
  },

  props: {
    model: {
      type: Boolean,
      required: false,
    },
    render: {
      type: Object,
      required: false,
    },
  },
  emits: ["closeDrag", "sortBtn"],
  methods: {
    log(evt) {
      if (evt.moved && this.length) {
        this.$emit(
          "sortBtn",
          this.line,
          evt.moved.newIndex + "",
          evt.moved.element.id
        );
      }
      if (evt.added && this.length) {
        this.$emit(
          "sortBtn",
          this.line,
          evt.added.newIndex + "",
          evt.added.element.id
        );
      }
      window.console.log(evt);
    },
    getLineId(line) {
      this.line = line.id;
    },
    close() {
      this.$emit("closeDrag");
    },
    checkLength(evt) {
      if (evt.path[1].childNodes.length > 10) {
        this.length = false;
      } else {
        this.length = true;
      }
    },
  },
  setup() {
    return {
      line: ref(null),
      sort: ref(null),
      id: ref(null),
      info: ref(false),
      length: ref(true),
    };
  },
  watch: {},
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
  &.wh_auto {
    width: auto;
    height: auto;
  }
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
.info {
  width: 22px;
  height: 22px;
  margin: 7px;
  border-radius: 50%;
  background-size: cover;
  background-image: url("../assets/info.svg");
  cursor: pointer;
  &_content {
    padding: 7px;
    max-width: 500px;
    font-size: 14px;
  }
}
.drag_container {
  padding: 5px;
}
.my-button {
  width: 28px;
  height: 28px;
  margin: 3px;
  flex-grow: 1;
}
.list-group {
  min-height: 20px;
  display: flex;
  transition: 0.3s all;
}
.sortable-ghost {
  opacity: 0.5;
}
.list-group-item i {
  cursor: pointer;
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
@media (max-width: 400px) {
  .header {
    font-size: 18px;
  }
}
</style>
