<template>
  <div class="i-no-cards pt-5 d-block mb-5">
    <p class="text-muted">Sin tareas.</p>
    <a href="#" style="text-decoration: none" @click="$emit('onNewTask')"
      >¿Porqué no comenzamos agregando algunas?</a
    >
  </div>
</template>

<script>
export default {
  name: "INoTasksContainer",
};
</script>

<style>
</style>