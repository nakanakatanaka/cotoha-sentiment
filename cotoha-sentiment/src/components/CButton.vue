<template>
  <button
    class="button"
    :disabled="disabled"
    :class="{'button--disabled': disabled}"
    @click="clickHandler"
  >
    <slot />
  </button>
</template>

<script>
export default {
  props: {
    disabled: Boolean
  },
  methods: {
    clickHandler(e) {
      this.$emit("click", e);
    }
  }
};
</script>

<style scoped>
.button {
  background-color: #ffffff;
  border-radius: 5px;
  border: 2px solid black;
}
.button:hover {
  cursor: pointer;
}
.button--disabled {
  opacity: 0.7;
}
.button--disabled:hover {
  cursor: not-allowed;
}
</style>