<template>
  <div class="app">
    <h1>Kanban Board</h1>
    <Board />
  </div>
</template>

<script>
import Board from './components/Board.vue';

export default {
  name: 'App',
  components: { Board }
};
</script>

<style>
.app {
  padding: 20px;
  font-family: Arial, sans-serif;
}
</style>