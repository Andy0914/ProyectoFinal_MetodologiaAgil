<template>
  <div id="app" class="container">
    <TaskModal @onSubmit="onSubmitTask" :tasks="tasks" ref="taskModal" />
    <div class="row mb-5">
      <div class="offset-sm-1 col-sm-5 text-left mb-3">
        <h2>Pomodoro</h2>
      </div>
      <div class="col-sm-5 offset-sm-1 mb-3 text-right">
        <b-button
          class="i-nueva-tarea shadow"
          variant="primary"
          @click="attemptNewTask"
          >Nueva tarea</b-button
        >
      </div>
    </div>
    <div class="container">
      <Draggable
        v-model="tasks"
        @start="onDragStart"
        @end="onDragEnd"
        class="row"
        v-if="tasks && tasks.length > 0"
      >
        <div
          v-for="(task, idx) of tasks"
          :key="idx"
          class="col-sm-3 offset-sm-1 mb-5"
        >
          <Card :task="task" />
        </div>
      </Draggable>
      <div class="i-no-cards pt-5 d-block" v-else>
        <p class="text-muted">Sin tareas.</p>
        <a href="#" style="text-decoration: none" @click="attemptNewTask"
          >¿Porqué no comenzamos agregando algunas?</a
        >
      </div>
    </div>
  </div>
</template>

<script>
import Card from "./components/Card";
import TaskModal from "./modals/Task";
import Draggable from "vuedraggable";

export default {
  name: "App",
  components: { Card, TaskModal, Draggable },
  data() {
    return {
      drag: false,
      tasks: [],
    };
  },
  methods: {
    onDragStart() {
      this.drag = true;
    },
    onDragEnd() {
      const { tasks } = this;
      this.drag = false;
      for (let i = 0; i < tasks.length; i++) {
        tasks[i].orderIdx = i;
      }
      localStorage.setItem("tasks", JSON.stringify(tasks));
    },
    attemptNewTask() {
      this.$refs.taskModal.$show(null);
    },
    onSubmitTask(task) {
      const { tasks } = this;
      task = JSON.parse(JSON.stringify(task));
      const idx = tasks.findIndex((e) => e.id === task.id);
      if (idx !== -1) {
        tasks[idx] = task;
      }
      tasks.push(task);
      localStorage.setItem("tasks", JSON.stringify(tasks));
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

.i-nueva-tarea {
}
</style>
