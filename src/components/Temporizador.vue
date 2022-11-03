<template>
  <div class="i-temporizador">
    <h4>{{ render.text }}</h4>
    <b-progress height="1rem" :value="render.value" />
  </div>
</template>

<script>
export default {
  name: "ITemporizador",
  props: {
    data: {
      type: Object,
      default() {
        return {
          totalSeconds: 0,
          remainingSeconds: 0,
        };
      },
    },
  },
  computed: {
    render() {
      const { totalSeconds, remainingSeconds } = this.data;
      const minutes = Math.floor(remainingSeconds / 60);
      const seconds = remainingSeconds - minutes * 60;
      const text = `${minutes.toString().padEnd(2, "0")}:${seconds
        .toString()
        .padEnd(2, "0")}`;
      const value = (remainingSeconds / totalSeconds) * 100;
      return { text, value };
    },
  },
};
</script>

<style>
</style>