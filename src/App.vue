<template>
  <div class="container">
    <the-select
      :specialties="specialties"
      :isError="isError"
      @error="isError = $event"
      @select-spec="selectedSpec = $event"
      @set-search="searchString = $event"
    ></the-select>
    <the-button
      class="search-button"
      :selectedSpec="selectedSpec"
      :searchString="searchString"
      @error="isError = true"
    >
      <span class="search-button__text">Найти</span>
      <span class="search-button__icon">
        <search-icon/>
      </span>
    </the-button>
  </div>
</template>

<script>
import TheSelect from '@/components/TheSelect.vue';
import TheButton from '@/components/TheButton.vue';
import SearchIcon from '@/assets/svg/search.svg?inline';

export default {
  name: 'App',

  components: {
    TheSelect,
    TheButton,
    SearchIcon,
  },

  data() {
    return {
      specialties: null,
      selectedSpec: null,
      searchString: '',
      isError: false,
    };
  },

  async created() {
    const res = await fetch('/specialties-list.json');
    const specialties = await res.json();
    this.specialties = specialties.data;
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

.container {
  display: flex;
  align-items: center;
  margin: 0 auto;
  padding: 20px;
  max-width: 600px;

  @media (max-width: 600px) {
    padding: 20px 10px;
    width: 100%;
  }
}

.search-button {
  margin-left: 10px;

  @media (max-width: 600px) {
    .search-button__text {
      display: none;
    }

    .search-button__icon {
      display: block;
    }
  }

  &:hover {
    svg {
      fill: $prim-bg-color;
    }
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
