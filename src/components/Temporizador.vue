<template>
  <div class="i-temporizador">
    <h2 v-if="data.isDescanso" class="text-primary fw-bold">Descanso</h2>
    <h4>{{ render.text }}</h4>
    <b-progress height="1rem" :value="render.value" :variant="data.isPause ? 'warning' : (data.isDescanso ? 'info' : 'primary')" />
    <b-alert class="mt-3 mb-0" variant="warning" :show="!isTemporizadorRunning && data.remainingSeconds > 0">El temporizador se ha puesto en pausa.</b-alert>
    <b-btn 
    class="mt-3"
    :variant="isTemporizadorRunning ? 'warning' : 'success'"
    v-if="data.remainingSeconds > 0"
    @click="$emit('onResumePause')"
    >
    {{ isTemporizadorRunning ? 'Pausar' : 'Reanudar'}} temporizador
    </b-btn>
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
          isPause: false,
          isDescanso: false,
        };
      },
    },
  },
  computed: {
    isTemporizadorRunning(){
      const { remainingSeconds, isPause } = this.data;
      return remainingSeconds > 0 && !isPause;
    },
    render() {
      const { totalSeconds, remainingSeconds } = this.data;
      const minutes = Math.floor(remainingSeconds / 60);
      const seconds = remainingSeconds - minutes * 60;
      const text = `${minutes.toString().padStart(2, "0")}:${seconds
        .toString()
        .padStart(2, "0")}`;
      const value = (remainingSeconds / totalSeconds) * 100;
      return { text, value };
    },
  },
};
</script>

<style>
</style>