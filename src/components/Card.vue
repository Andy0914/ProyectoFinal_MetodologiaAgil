<template>
  <div class="i-card-container">
    <div
      class="i-card shadow rounded pt-3"
      @click="$emit('click')"
      :class="[$classes]"
    >
      <h4 class="mb-3">{{ task.name }}</h4>
      <small>{{ task.description }}</small>
    </div>
  </div>
</template>

<script>
export default {
  name: "ICard",
  props: {
    task: {
      type: Object,
      default() {
        return null;
      },
    },
    variant: {
      type: String,
      default: "",
    },
  },
  computed: {
    $classes() {
      const { variant } = this;
      const classes = [];
      if (variant === "progress") classes.push("ic-progress");

      return classes;
    },
  },
};
</script>

<style>
.i-card-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}
.i-card {
  cursor: pointer;
  user-select: none;
  width: 150px;
  min-height: 150px;
  padding-bottom: 1rem;
  display: block;
  background-color: #fff80a;
  overflow: hidden;
  transition: 200ms;
}

.i-card.ic-progress {
  background-color: #ff7f3f;
}
</style>