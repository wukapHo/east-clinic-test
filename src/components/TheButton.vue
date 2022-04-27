<template>
  <button
    class="button"
    @click="goToURL(selectedSpec, searchString)"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'TheButton',

  props: {
    selectedSpec: {
      type: Object,
      default() {
        return {};
      },
    },

    searchString: {
      type: String,
      default: '',
    },
  },

  methods: {
    goToURL(item, searchStr) {
      if (!item && !searchStr) {
        this.$emit('error');
        return;
      }

      if (item) {
        console.log('redirect to: ', item.url);
      } else {
        console.log('redirect to search page with query: ', searchStr);
      }
    },
  },
};
</script>

<style lang="scss">
.button {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px 25px;
  background: $prim-bg-color;
  border: 1px solid $prim-bg-color;
  border-radius: 10px;
  color: $second-bg-color;
  cursor: pointer;
  z-index: 10;
  transition: 0.3s;

  @media (max-width: 600px) {
    padding: 5px;
  }

  &:hover {
    background: $second-bg-color;
    color: $prim-bg-color;
  }

  &:active {
    outline: 1px solid $prim-bg-color;
  }
}
</style>
