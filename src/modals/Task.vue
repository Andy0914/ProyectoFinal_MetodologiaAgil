<template>
  <b-modal
    id="task-modal"
    ref="modal"
    v-model="show"
    @ok="onSubmit"
    ok-title="Guardar"
    cancel-title="Cerrar"
    cancel-variant="outline-dark"
    hide-header
    hide-header-close
  >
    <h4 class="pt-3 mb-3">Crear tarea</h4>
    <form @submit="onSubmit">
      <p class="mb-1">Nombre</p>
      <input
        type="text"
        class="form-control mb-3"
        v-model="form.name"
        required
        min="1"
        maxlength="100"
        autofocus
      />
      <p class="mb-1">Descripción</p>
      <textarea
        class="form-control i-description"
        v-model="form.description"
        rows="3"
      ></textarea>
      <input type="submit" class="d-none" />
    </form>
  </b-modal>
</template>

<script>
export default {
  name: "TaskModal",
  props: {
    tasks: {
      type: Object,
      default() {
        return { queued: [], progress: [], finished: [] };
      },
    },
  },
  data() {
    return {
      show: false,
      form: {
        id: 0,
        orderIdx: 0,
        name: "",
        description: "",
        status: "queued",
        finishedAt: 0,
      },
    };
  },
  computed: {
    $error() {
      const { name, status } = this.form;
      if (!name || name.length <= 0) return "Debe escribir un nombre.";

      const tasks = this.tasks[status];
      if (tasks && tasks.length) {
        for (const task of tasks) {
          if (task.name.toLowerCase() === name.toLowerCase()) {
            return "El nombre de esta tarea ya se encuentra en uso.";
          }
        }
      }

      return "";
    },
  },
  methods: {
    $show(task = null) {
      const valid = typeof task === "object" && task !== null;
      const status = valid ? task.status : "queued";
      const tasks = this.tasks[status];
      const frm = this.form;
      frm.id = valid ? task.id : new Date().getTime();
      frm.orderIdx = valid ? task.orderIdx : tasks.length;
      frm.name = valid ? task.name : "";
      frm.description = valid ? task.description : "";
      frm.status = status;
      this.show = true;
    },
    $hide() {
      this.show = false;
    },
    onSubmit(e) {
      if (e !== null && e !== undefined) e.preventDefault();
      if (this.$error) {
        this.$swal({
          icon: "error",
          title: "Error",
          text: this.$error,
        });
        return;
      }
      this.$emit("onSubmit", { ...this.form });
      this.$hide();
    },
  },
};
</script>

<style>
.i-description {
  resize: none;
}
</style>