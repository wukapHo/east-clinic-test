<template>
  <div class="select">
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
        class="select__input"
        v-model="search"
        @click.stop
        @input="setSearchString(search)"
      >
      <label class="select__placeholder" for="spec">Врач, заболевания, услуги</label>
      <button
        v-show="isActive && search"
        class="select__input-clear"
        @click.stop="clearSearch"
      ></button>
    </div>

    <div
      v-if="isActive"
      class="select__options"
    >
      <p class="select__options-title">Специальность</p>
      <p
        v-if="filteredSpecialties.length === 0"
        class="select__options-error"
      >
        Не найдено специальностей
      </p>
      <ul class="select__options-list">
        <li
          v-for="item in filteredSpecialties"
          :key="item.id"
          tabindex="0"
          class="select__options-item"
          @click="selectSpec(item)"
          @keypress.enter="selectSpec(item)"
        >
          {{ item.title }}
        </li>
      </ul>
    </div>

  </div>
  <div
    v-if="isActive"
    class="overlay"
    @click="isActive = !isActive"
  ></div>
</template>

<script>
export default {
  name: 'TheSelect',

  props: {
    specialties: {
      type: Array,
      require: true,
    },

    isError: {
      type: Boolean,
      default: false,
    },
  },

  emits: {
    'select-spec': null,
    error: null,
    'set-search': null,
  },

  data() {
    return {
      isActive: false,
      search: '',
      selectedSpec: null,
    };
  },

  computed: {
    filteredSpecialties() {
      return this.specialties.filter((item) => item.title
        .toLowerCase().includes(this.search.toLowerCase()));
    },
  },

  watch: {
    isError() {
      if (this.isError) {
        this.toggleActivity();
      }
    },
  },

  methods: {
    toggleActivity() {
      this.isActive = !this.isActive;
      this.$nextTick().then(() => {
        this.$refs.input.focus();
      });
    },

    selectSpec(item) {
      this.search = item.title;
      this.selectedSpec = item;
      this.isActive = !this.isActive;
      this.$emit('select-spec', this.selectedSpec);
      this.$emit('error', false);
    },

    clearSearch() {
      this.search = '';
      this.selectedSpec = null;
      this.$emit('select-spec', this.selectedSpec);
      this.$emit('error', false);
    },

    setSearchString(search) {
      this.$emit('set-search', search);
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
    z-index: 10;
    transition: 0.3s;

    &--active {
      background: url('../assets/svg/arrow-up.svg') no-repeat right 20px center $second-bg-color;
      border: 1px solid #1160ff;

      .select__placeholder,
      .select__input:not(:placeholder-shown) ~ .select__placeholder {
        font-size: 11px;
        line-height: 16px;
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

  &__options {
    position: absolute;
    top: 65px;
    display: block;
    width: 100%;
    height: 270px;
    background: $second-bg-color;
    box-shadow: 0px 14px 24px rgba(31, 49, 73, 0.25);
    border-radius: 5px;
    overflow-y: scroll;
    z-index: 10;
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
    position: sticky;
    top: 0;
    background: $second-bg-color;
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

    &:hover {
      background: $third-bg-color;
    }
  }
}

.overlay {
  display: block;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 5;
}
</style>
