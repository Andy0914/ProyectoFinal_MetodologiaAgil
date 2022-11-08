<template>
  <div class="i-temporizador">
    <h2 v-if="data.isDescanso" class="text-primary fw-bold">Descanso</h2>
    <b>{{ $pomodorosRestantes }}</b>
    <h4>{{ render.text }}</h4>
    <b-progress
      height="1rem"
      :value="render.value"
      :variant="data.isPause ? 'warning' : data.isDescanso ? 'info' : 'primary'"
    />
    <b-alert
      class="mt-3 mb-0"
      variant="warning"
      :show="!isTemporizadorRunning && data.remainingSeconds > 0"
      >El temporizador se ha puesto en pausa.</b-alert
    >
    <b-btn
      class="mt-3 d-inline-block mr-3"
      :variant="isTemporizadorRunning ? 'warning' : 'success'"
      v-if="data.remainingSeconds > 0"
      @click="$emit('onResumePause')"
    >
      {{ isTemporizadorRunning ? "Pausar" : "Reanudar" }} temporizador
    </b-btn>
    <b-btn
      class="mt-3 d-inline-block"
      variant="dark"
      v-if="data.remainingSeconds > 0 && data.isDescanso"
      @click="onOmitirDescanso"
    >
      Omitir descanso
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
    $pomodorosRestantes() {
      const { count, descansoLargo } = this.data;
      const restantes = descansoLargo - count;
      if (restantes > 1) {
        return `${restantes} pomodoros restantes`;
      }
      return `${restantes} pomodoro restante`;
    },
    isTemporizadorRunning() {
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
  methods: {
    async onOmitirDescanso() {
      const result = await this.$swal({
        title: "¿Desea omitir el descanso?",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Sí",
        cancelButtonText: "No",
        reverseButtons: true,
      });
      if (!result || !result.value) return;

      this.$emit("onOmitirDescanso");
    },
  },
};
</script>

<style>
.mr-3 {
  margin-right: 1rem;
}
</style>