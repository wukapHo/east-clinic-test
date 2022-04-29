<template>
  <div class="select" @click.stop>
    <div
      class="select__picker"
      :class="{
        'select__picker--active': isActive,
        'select__picker--error': isError,
      }"
      @click="toggleActivity"
    >
      <input
        v-show="isActive"
        type="text"
        placeholder=" "
        ref="input"
        id="spec"
        autocomplete="off"
        class="select__input"
        v-model="search"
        @click.stop
        @input="setSearchString(search)"
      >
      <label
        v-if="isPlaceholderShow"
        class="select__placeholder"
        for="spec"
      >
        {{ this.placeholder }}
      </label>

      <p v-else>{{ this.selecterContent }}</p>

      <button
        v-show="isActive && search"
        class="select__input-clear"
        @click.stop="clearInput"
      />
    </div>

    <p v-if="isError" class="select__error">Обязательное поле</p>

    <transition name="move">
      <div v-show="isActive" class="select__options">
        <p v-if="filteredClinicData.length === 0" class="select__options-error">
          {{ this.emptyListPlaceholder }}
        </p>

        <ul
          v-for="configItem in config"
          :key="configItem.type"
          ref="list"
          class="select__options-list"
        >
          <li
            v-if="filteredClinicData.filter((elem) => elem.type === configItem.type).length"
            class="select__options-title"
          >
            {{ configItem.title }}
          </li>
          <li
            v-for="(item, i) in filteredClinicData.filter((elem) => elem.type === configItem.type)"
            :key="item.id"
            tabindex="0"
            class="select__options-item"
            :class="{
              'select__options-item--hovered': this.cursor === this.filteredClinicData.indexOf(item)
            }"
            @click="selectClinicItem(item)"
            @mouseover="cursor = i"
          >
            {{ item.title }}
          </li>
        </ul>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'TheSearchBox',

  props: {
    rawData: {
      type: Array,
      require: true,
    },

    config: {
      type: Array,
      required: true,
    },

    placeholder: {
      type: String,
      default: '',
    },

    emptyListPlaceholder: {
      type: String,
      default: '',
    },

    isError: {
      type: Boolean,
      default: false,
    },
  },

  emits: ['select', 'error', 'input'],

  data() {
    return {
      isActive: false,
      search: '',
      selectedItem: {},
      curentItem: {},
      cursor: -1,
      itemsForView: 7,
    };
  },

  computed: {
    sortedClinicData() {
      let list = [];
      this.config.forEach((item) => {
        list = [...list, ...this.rawData.filter(({ type }) => type === item.type)];
      });

      return list;
    },

    normalizeSearchStr() {
      return this.search.toLowerCase();
    },

    filteredClinicData() {
      return this.sortedClinicData.filter(({ title }) => title
        .toLowerCase().includes(this.normalizeSearchStr));
    },

    selecterContent() {
      return this.search || this.selectedItem.title;
    },

    isPlaceholderShow() {
      if (this.selecterContent && !this.isActive) {
        return false;
      }

      return true;
    },

    elementsForScroll() {
      let itemList = [];

      this.$refs.list.forEach((item) => {
        itemList = [...itemList, ...item.getElementsByClassName('select__options-item')];
      });

      return itemList;
    },
  },

  watch: {
    isError() {
      if (this.isError) {
        this.toggleActivity();
      }
    },

    isActive() {
      if (this.isActive) {
        document.body.addEventListener('click', this.toggleActivity);
        document.body.addEventListener('keydown', this.navigateFromArrow);
      } else {
        document.body.removeEventListener('click', this.toggleActivity);
        document.body.removeEventListener('keydown', this.navigateFromArrow);
      }
    },
  },

  methods: {
    toggleActivity() {
      this.isActive = !this.isActive;
      if (this.isActive) {
        this.$nextTick().then(() => {
          this.$refs.input.focus();
        });
      } else {
        this.cursor = -1;
      }
    },

    itemView(item) {
      if (item && item.scrollIntoView) {
        item.scrollIntoView(false);
      }
    },

    keyUp() {
      if (this.cursor > -1) {
        this.cursor -= 1;
        this.cursor = this.cursor < 0 ? this.filteredClinicData.length - 1 : this.cursor;
        this.itemView(this.elementsForScroll[this.cursor]);
      }
    },

    keyDown() {
      if (this.cursor < this.filteredClinicData.length) {
        this.cursor += 1;
        this.cursor = this.cursor >= this.filteredClinicData.length ? 0 : this.cursor;
        this.itemView(this.elementsForScroll[this.cursor]);
      }
    },

    keyEnter() {
      if (this.currentItem) {
        this.selectClinicItem(this.currentItem);
      }
    },

    keyBackspace() {
      this.selectedItem = {};
      this.$emit('select', this.selectedItem);
    },

    navigateFromArrow(event) {
      if (event.code === 'ArrowUp') {
        this.keyUp();
      } else if (event.code === 'ArrowDown') {
        this.keyDown();
      } else if (event.code === 'Enter' || event.code === 'Space') {
        this.keyEnter();
      } else if (event.code === 'Backspace') {
        this.keyBackspace();
      }

      this.currentItem = this.filteredClinicData[this.cursor];
    },

    selectClinicItem(item) {
      this.search = item.title;
      this.selectedItem = item;
      this.isActive = !this.isActive;
      this.$emit('select', this.selectedItem);
      this.$emit('error', false);
    },

    clearInput() {
      this.search = '';
      this.selectedItem = {};
      this.$nextTick(() => {
        this.$refs.input.focus();
      });
      this.$emit('select', this.selectedItem);
      this.$emit('error', false);
    },

    setSearchString(search) {
      this.$emit('input', search);
      this.$emit('error', false);
    },
  },
};
</script>

<style lang="scss">
.select {
  position: relative;
  width: 100%;

  &__picker {
    z-index: 10;
    display: flex;
    flex-direction: column-reverse;
    align-items: left;
    justify-content: center;
    position: relative;
    padding: 0 40px 0 20px;
    width: 100%;
    height: 54px;
    background: url('../assets/svg/arrow-down.svg') no-repeat right 20px center $third-bg-color;
    border: 1px solid transparent;
    border-radius: 10px;
    cursor: pointer;
    transition: 0.3s;

    &--active {
      background: url('../assets/svg/arrow-up.svg') no-repeat right 20px center $second-bg-color;
      border: 1px solid #1160ff;

      .select__placeholder,
      .select__input:not(:placeholder-shown) ~ .select__placeholder {
        font-size: 11px;
        letter-spacing: 0.3px;
        color: #9f9f9f;
      }
    }

    &--error {
      border: 1px solid #f70a0a;
    }
  }

  &__placeholder {
    font-size: 16px;
    line-height: 24px;
    color: #757f93;
    transition: font-size 0.3s;

    @media (max-width: 600px) {
      font-size: 14px;
    }
  }

  &__input {
    display: block;
    width: 90%;
    border: none;
    outline: none;
    font-size: 16px;
    line-height: 24px;
    color: #1f3149;

    @media (max-width: 600px) {
      line-height: 22px;
    }
  }

  &__input-clear {
    position: absolute;
    right: 40px;
    width: 24px;
    height: 24px;
    border: none;
    background: url('../assets/svg/cross.svg') no-repeat center;
    cursor: pointer;
    opacity: 0.7;
    transition: 0.3s;

    &:hover {
      opacity: 1;
    }
  }

  &__error {
    position: absolute;
    margin-top: 5px;
    padding: 0 20px;
    font-size: 14px;
    line-height: 16px;
    color: #f70a0a;
  }

  &__options {
    z-index: 10;
    display: block;
    width: 100%;
    height: 270px;
    position: absolute;
    top: 75px;
    background: $second-bg-color;
    box-shadow: 0px 14px 24px rgba(31, 49, 73, 0.25);
    border-radius: 5px;
    overflow-y: scroll;
    // Scrollbar for FireFox
    scrollbar-width: auto;
    scrollbar-color: $prim-bg-color #f1f4f9;

    // Scrollbar for Chrome/Edge/Safari
    &::-webkit-scrollbar {
      width: 5px;
      height: 10px;
    }

    &::-webkit-scrollbar-track {
      background: #f1f4f9;
    }

    &::-webkit-scrollbar-thumb {
      background-color: $prim-bg-color;
      border-radius: 5px;
    }
  }

  &__options-title,
  &__options-error {
    padding: 5px 14px;
    font-size: 13px;
    line-height: 19px;
    letter-spacing: 0.2px;
    color: #808080;
    cursor: default;
  }

  &__options-title {
    padding-left: 10px;
    border-bottom: 1px solid #9cbdff;
  }

  &__options-item {
    padding: 5px 14px;
    color: #1f3149;
    font-size: 14px;
    line-height: 20px;
    letter-spacing: 0.1px;
    cursor: pointer;
    transition: 0.3s;

    &:hover,
    &--hovered {
      background: $third-bg-color;
    }
  }
}

.move-enter-active,
.move-leave-active {
  transition: 0.3s;
}

.move-enter-to,
.move-leave-from {
  opacity: 1;
}

.move-enter-from,
.move-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}
</style>
