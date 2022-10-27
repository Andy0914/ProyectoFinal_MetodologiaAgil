<template>
  <div class="i-card-container">
    <div class="i-card shadow rounded pt-3">
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
  width: 150px;
  height: 150px;
  display: block;
  background-color: #fff80a;
  overflow: hidden;
}
</style>