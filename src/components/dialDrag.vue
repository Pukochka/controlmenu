<template>
  <p-dialog :model="model" class="">
    <p-card class="max">
      <div class="flex justify-between">
        <div class="header">Перемещение кнопок</div>
        <div class="info" @click="info = !info"></div>
      </div>
      <p-separator />
      <div class="drag_container" v-for="(line, index) in render" :key="index">
        <div
          class=""
          v-for="(btns, btnindex) in line"
          :key="btnindex"
          @dragover="getLineId(btns)"
          @touchend="getLineId(btns)"
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
                {{ threeLetters(element.data.text) }}
              </div>
            </template>
          </draggable>
        </div>
      </div>
      <div class="actions">
        <div class="button line" @click="close">
          <div class="button_helper"></div>
          Закрыть
        </div>
      </div>
    </p-card>
  </p-dialog>
  <p-dialog :model="info">
    <p-card>
      <div class="header">Помощь по перемещению</div>
      <p-separator />
      <p class="info_content">
        Для того чтобы переместить кнопку, просто наведите на неё и зажмите
        левую кнопку мыши, затем переместите на нужное место
      </p>
      <div class="button line" @click="info = !info">
        <div class="button_helper"></div>
        Закрыть
      </div>
    </p-card>
  </p-dialog>
</template>
<script>
  import { ref } from 'vue';
  import pCard from './pCard.vue';
  import pDialog from './pDialog.vue';
  import pSeparator from './pSeparator.vue';
  import draggable from 'vuedraggable';

  export default {
    components: {
      'p-dialog': pDialog,
      'p-card': pCard,
      'p-separator': pSeparator,
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
    emits: ['closeDrag', 'sortBtn'],
    methods: {
      log(evt) {
        if (evt.moved) {
          console.log(this.line);
          this.$emit(
            'sortBtn',
            this.line,
            evt.moved.newIndex,
            evt.moved.element.id
          );
        }
        if (evt.added) {
          console.log(this.line);
          this.$emit(
            'sortBtn',
            this.line,
            evt.added.newIndex + '',
            evt.added.element.id
          );
        }
        window.console.log(evt);
      },
      getLineId(line) {
        this.line = line.id;
      },
      close() {
        this.$emit('closeDrag');
      },
      threeLetters(word) {
        return [...word].splice(0, 3).join('') + '..';
      },
    },
    setup() {
      return {
        line: ref(null),
        sort: ref(null),
        id: ref(null),
        info: ref(false),
      };
    },
    watch: {},
  };
</script>
<style lang="scss" scoped>
  .header {
    font-weight: 500;
    padding: 7px;
    font-size: 18px;
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
  .info {
    width: 26px;
    height: 26px;
    margin: 7px;
    background-size: cover;
    background-image: url('../assets/info.svg');
    &_content {
      padding: 10px;
      max-width: 500px;
    }
  }
  .drag_container {
    padding: 5px;
  }
  .my-button {
    width: 28px;
    height: 28px;
    margin: 3px;
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
</style>
