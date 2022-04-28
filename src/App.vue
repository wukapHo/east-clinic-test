<template>
  <div class="container">
    <the-search-box
      :placeholder="placeholder"
      :empty-list-placeholder="emptyListPlaceholder"
      :raw-data="rawClinicData"
      :config="config"
      :is-error="isError"
      :class="{ 'error': isError }"
      @error="isError = $event"
      @select="selectedItem = $event"
      @input="searchString = $event"
    />

    <button class="button" @click.stop="goToURL(selectedItem, searchString)">
      <span class="button__text">Найти</span>
      <span class="button__icon">
        <search-icon/>
      </span>
    </button>
  </div>
</template>

<script>
import TheSearchBox from '@/components/TheSearchBox.vue';
import SearchIcon from '@/assets/svg/search.svg?inline';

export default {
  name: 'App',

  components: {
    TheSearchBox,
    SearchIcon,
  },

  data() {
    return {
      rawClinicData: [],
      config: [
        {
          title: 'Специальность',
          type: 'spec',
        },
      ],
      placeholder: 'Врач, заболевания, услуги',
      emptyListPlaceholder: 'Не найдено специальностей',
      selectedItem: {},
      searchString: '',
      isError: false,
    };
  },

  async created() {
    this.rawClinicData = await this.getClinicData();
  },

  methods: {
    async getClinicData() {
      const res = await fetch('/mock/data-list.json');
      const data = await res.json();

      return data.data;
    },

    isValid(data) {
      if (!data.item.title && !data.searchStr) {
        this.isError = true;
        return false;
      }

      return true;
    },

    goToURL(item, searchStr) {
      if (!this.isValid({ item, searchStr })) {
        return;
      }

      if (item.title) {
        console.log('redirect to: ', item.url);
      } else {
        console.log('redirect to search page with query: ', searchStr);
      }
    },
  },
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Open Sans', sans-serif;
  font-weight: 400;
  list-style: none;
}

html,
body {
  height: 100%;
}

.container {
  display: flex;
  align-items: center;
  margin: 0 auto;
  padding: 20px;
  max-width: 600px;

  @media (max-width: $mob-width) {
    padding: 20px 10px;
    width: 100%;
  }
}

.button {
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 10px;
  padding: 15px 25px;
  background: $prim-bg-color;
  border: 1px solid $prim-bg-color;
  border-radius: 10px;
  color: $second-bg-color;
  cursor: pointer;
  transition: 0.3s;

  @media (max-width: $mob-width) {
    padding: 9px;

    .button__text {
      display: none;
    }

    .button__icon {
      display: block;
    }
  }

  &:hover {
    background: $second-bg-color;
    color: $prim-bg-color;

    svg {
      fill: $prim-bg-color;
    }
  }

  &:active {
    outline: 1px solid $prim-bg-color;
  }

  &__text {
    font-weight: 600;
    font-size: 17px;
    line-height: 20px;
  }

  &__icon {
    display: none;

    svg {
      fill: $second-bg-color;
    }
  }
}
</style>
