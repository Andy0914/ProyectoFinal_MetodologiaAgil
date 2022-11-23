<template>
  <div
    class="i-card-container"
    @mouseover="isHover = true"
    @mouseleave="isHover = false"
  >
    <div
      class="i-card shadow rounded pt-3"
      @click="$emit('click')"
      :class="[$classes]"
    >
      <h4 class="mb-3">{{ task.name }}</h4>
      <div class="i-card-finished" v-if="$finishedAt">
        <b>Terminado</b>
        <b>{{ $finishedAt }}</b>
      </div>
      <small v-else>{{ task.description }}</small>
    </div>
    <div class="d-flex">
      <b-btn
        v-if="task.status !== 'finished'"
        size="sm"
        variant="danger"
        class="i-card-btn d-block mt-3 mr-3"
        :class="{ hover: isHover }"
        @click="$emit('onDelete', task)"
        >Eliminar</b-btn
      >
      <b-btn
        v-if="task.status !== 'finished'"
        size="sm"
        variant="dark"
        class="i-card-btn d-block mt-3"
        :class="{ hover: isHover }"
        @click="$emit('onEdit', task)"
        >Modificar</b-btn
      >
    </div>
  </div>
</template>

<script>
import moment from "moment";

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
  data() {
    return {
      isHover: false,
    };
  },
  computed: {
    $classes() {
      const { variant } = this;
      const classes = [];
      if (variant === "progress") classes.push("ic-progress");
      if (variant === "finished") classes.push("ic-finished");

      return classes;
    },
    $finishedAt() {
      const { finishedAt, status } = this.task;
      if (status !== "finished") return null;

      const $finishedAt = new moment(finishedAt);
      return $finishedAt.format("DD/MM/YYYY LTS");
    },
  },
};
</script>

<style>
.i-card-container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
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

.i-card.ic-finished {
  background-color: #82cd47;
}

.i-card-finished {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  color: rgba(255, 255, 255, 0.8);
}

.i-card-btn {
  opacity: 0;
  transition: 200ms;
}

.i-card-btn.hover {
  opacity: 1;
}
</style>