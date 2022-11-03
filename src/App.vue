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
          >Cancelar temporizador</b-button
        >
      </div>
    </div>
    <div class="row mb-5">
      <div class="col-sm-8 offset-sm-2">
        <Temporizador :data="temporizador" />
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
      },
    };
  },
  computed: {
    isTemporizadorRunning() {
      return this.temporizador.idx > 0;
    },
  },
  methods: {
    onDragStart() {
      this.drag = true;
    },
    onDragEnd() {
      this.drag = false;
    },
    onDragChange() {
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
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
      console.log({ task });
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
      temporizador.idx = setInterval(() => {
        temporizador.remainingSeconds--;
        if (temporizador.remainingSeconds <= 0) {
          clearInterval(temporizador.idx);
          temporizador.idx = 0;
        }
      }, 1000);
    },
    cancelarTemporizador() {
      const { temporizador } = this;
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
