<template>
  <div id="app" class="container">
    <TaskModal @onSubmit="onSubmitTask" :tasks="tasks" ref="taskModal" />
    <div class="row mb-5">
      <div class="offset-sm-1 col-sm-4 text-left mb-3">
        <h2>Pomodoro</h2>
      </div>
      <div class="col-sm-7 mb-3 text-right">
        <b-button class="shadow i-btn" variant="primary" @click="attemptNewTask"
          >Nueva tarea</b-button
        >
        <b-button
          class="shadow i-btn"
          variant="dark"
          @click="iniciarTemporizador"
          v-if="!isTemporizadorRunning && tasks.progress.length > 0"
          >Iniciar temporizador</b-button
        >
        <b-button
          class="shadow i-btn"
          variant="danger"
          @click="cancelarTemporizador"
          v-else-if="isTemporizadorRunning"
          >Reestablecer temporizador</b-button
        >
      </div>
    </div>
    <div class="row mb-5">
      <div class="col-sm-8 offset-sm-2">
        <Temporizador :data="temporizador" @onResumePause="onResumePause" />
      </div>
    </div>
    <h4 class="mb-5 d-block">Tareas pendientes</h4>
    <div class="container mb-5">
      <Draggable
        :list="tasks.queued"
        @start="onDragStart"
        @end="onDragEnd"
        @change="onDragChange"
        id="tasksQueued"
        group="status"
        class="row i-drag"
      >
        <div
          v-for="task of tasks.queued"
          :key="task.id"
          class="col-sm-3 offset-sm-1 mb-5"
        >
          <Card :task="task" @click="() => handleClickTask(task)" />
        </div>
        <NoTasksContainer v-if="!tasks.queued || tasks.queued.length <= 0" />
      </Draggable>
    </div>
    <h4 class="mb-5 d-block">Tareas en progreso</h4>
    <div class="container">
      <Draggable
        :list="tasks.progress"
        id="tasksProgress"
        @start="onDragStart"
        @end="onDragEnd"
        @change="onDragChange"
        group="status"
        class="row i-drag"
      >
        <div
          v-for="task of tasks.progress"
          :key="task.id"
          class="col-sm-3 offset-sm-1 mb-5"
        >
          <Card
            :task="task"
            @click="() => handleClickTask(task)"
            variant="progress"
          />
        </div>
        <NoTasksContainer
          v-if="!tasks.progress || tasks.progress.length <= 0"
        />
      </Draggable>
    </div>
    <h4 class="mb-5 d-block">Tareas terminadas</h4>
    <div class="container">
      <div :list="$finishedTasks" id="tasksFinished" group="status" class="row">
        <div
          v-for="task of $finishedTasks"
          :key="task.id"
          class="col-sm-3 offset-sm-1 mb-5"
        >
          <Card :task="task" variant="finished" />
        </div>
        <NoTasksContainer
          v-if="!tasks.finished || tasks.finished.length <= 0"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Card from "./components/Card";
import TaskModal from "./modals/Task";
import NoTasksContainer from "./components/NoTasksContainer";
import Temporizador from "./components/Temporizador";
import Draggable from "vuedraggable";

export default {
  name: "App",
  components: { Card, TaskModal, Draggable, NoTasksContainer, Temporizador },
  data() {
    return {
      drag: false,
      tasks: {
        queued: [],
        progress: [],
        finished: [],
      },
      temporizador: {
        totalSeconds: 0,
        remainingSeconds: 0,
        idx: 0,
        isPause: false,
        isDescanso: false,
        count: 0,
        descansoLargo: 4,
      },
      audioReference: null,
    };
  },
  watch: {
    remainingSeconds(value) {
      const { temporizador } = this;

      if (value === 5 && temporizador.totalSeconds > 0) {
        this.mostrarNotifacionDescanso();
      }

      if (value > 0 || temporizador.totalSeconds <= 0) return;

      const audio = new Audio(require("./assets/timer.mp3"));
      audio.play();

      this.audioReference = audio;
    },
  },
  computed: {
    isTemporizadorRunning() {
      return this.temporizador.idx > 0;
    },
    remainingSeconds() {
      return this.temporizador.remainingSeconds;
    },
    $finishedTasks() {
      const { finished } = this.tasks;
      return finished.sort((a, b) => b.finishedAt - a.finishedAt);
    },
  },
  methods: {
    async mostrarNotifacionDescanso() {
      const { temporizador } = this;
      const { isDescanso } = temporizador;
      try {
        await this.$swal({
          title: "El temporizador está por acabar",
          text: "",
          icon: "warning",
          confirmButtonText: "Entedido",
        });
      } catch (e) {
        console.log(e);
      }

      if (this.audioReference !== null) {
        this.audioReference.pause();
        this.audioReference = null;
      }

      clearInterval(temporizador.idx);
      temporizador.idx = 0;
      this.iniciarTemporizador();
      temporizador.isPause = false;
      temporizador.isDescanso = !isDescanso;
      temporizador.totalSeconds = 60 * 5;
      temporizador.remainingSeconds = temporizador.totalSeconds;
      if (temporizador.isDescanso) {
        temporizador.count++;
        if (temporizador.count >= temporizador.descansoLargo) {
          temporizador.totalSeconds = 20 * 60;
          temporizador.remainingSeconds = temporizador.totalSeconds;
          temporizador.count = 0;
        }
      }
    },
    onResumePause() {
      const { isPause } = this.temporizador;
      this.temporizador.isPause = !isPause;
    },
    onDragStart() {
      this.drag = true;
    },
    onDragEnd() {
      this.drag = false;
    },
    onDragChange(e) {
      const movedTask = e && e.added ? e.added.element : null;
      let lastStatus = movedTask ? movedTask.status : null;
      let newStatus = null;

      for (const [status, tasks] of Object.entries(this.tasks)) {
        let idx = 0;
        for (const task of tasks) {
          if (movedTask !== null && task.id === movedTask.id) {
            newStatus = status;
          }
          task.status = status;
          task.orderIdx = idx;
          idx++;
        }
      }

      localStorage.setItem("tasks", JSON.stringify(this.tasks));

      // Requerimiento historia 13: Marcar tarea en progreso como pendiente
      if (lastStatus === "progress" && newStatus === "queued") {
        this.$swal({
          title: "Se ha marcado la tarea como pendiente.",
          text: `La tarea "${movedTask.name}" se ha marcado como pendiente exitosamente.`,
          icon: "success",
          confirmButtonText: "Entendido",
        });
      }
    },
    attemptNewTask() {
      this.$refs.taskModal.$show(null);
    },
    onSubmitTask(task) {
      const tasks = this.tasks[task.status];
      task = JSON.parse(JSON.stringify(task));
      const idx = tasks.findIndex((e) => e.id === task.id);
      if (idx !== -1) {
        tasks[idx] = task;
      }
      tasks.push(task);
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },
    async handleClickTask(task) {
      const { tasks } = this;
      const idx = tasks.progress.findIndex((e) => e.id === task.id);
      if (idx === -1) {
        return;
      }

      if (task.status === "progress") {
        const result = await this.$swal({
          title: "¿Desea marcar esta tarea como terminada?",
          text: `La tarea "${task.name}" será marcada como terminada, no podrá deshacer esta acción.`,
          icon: "warning",
          showCancelButton: true,
          confirmButtonText: "Sí",
          cancelButtonText: "No",
          reverseButtons: true,
        });
        if (!result || !result.value) return;
      }

      tasks.progress.splice(idx, 1);
      tasks.finished.push(task);
      task.finishedAt = new Date().getTime();

      // Mapear persitencia
      this.onDragChange();

      this.$swal({
        title: "Se ha marcado la tarea como completada.",
        text: `La tarea "${task.name}" se ha marcado como completada exitosamente.`,
        icon: "success",
        confirmButtonText: "Entendido",
      });
    },
    iniciarTemporizador() {
      const tasks = this.tasks.progress;
      if (!tasks || !tasks.length) {
        this.$swal({
          icon: "error",
          title: "Error",
          text: "No puede iniciar el temporizador si no hay tareas pendientes.",
        });
      }

      const { temporizador } = this;
      temporizador.totalSeconds = 25 * 60;
      temporizador.remainingSeconds = temporizador.totalSeconds;
      temporizador.isPause = false;
      temporizador.idx = setInterval(() => {
        if (temporizador.isPause) {
          return;
        }

        temporizador.remainingSeconds--;
        if (temporizador.remainingSeconds <= 0) {
          clearInterval(temporizador.idx);
          temporizador.idx = 0;
        }
      }, 1000);
    },
    async cancelarTemporizador() {
      const { temporizador } = this;
      const result = await this.$swal({
        title: "¿Desea reestablecer el temporizador?",
        text: "El temporizador se resteblecerá a su inicio.",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Sí",
        cancelButtonText: "No",
        reverseButtons: true,
      });

      if (result.isConfirmed !== true) return;

      clearInterval(temporizador.idx);
      temporizador.idx = 0;
      temporizador.totalSeconds = 0;
      temporizador.remainingSeconds = 0;
    },
  },
  created() {
    const tasks = localStorage.getItem("tasks");
    if (typeof tasks === "string" && tasks.length) {
      this.tasks = JSON.parse(tasks);
    }
    // for (let i = 0; i < 12; i++) this.cards.push(i);
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.i-btn {
  margin-left: 1rem;
}

.i-drag {
  padding: 0.5rem;
}
</style>
