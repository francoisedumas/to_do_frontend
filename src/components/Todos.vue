<template>
  <div>
    <h3>Todos</h3>
    <div class="legend">
      <span>Click to mark as complete.</span>
      <span>
        <span class="incomplete-box">Incomplete</span>
      </span>
      <span>
        <span class="complete-box">Complete</span>
      </span>
    </div>
    <div class="todos">
      <div
        @click="onDoubleClick(todo)"
        v-for="todo in allTodos"
        :key="todo.id"
        class="todo"
        v-bind:class="{ 'is-complete': todo.completed }"
      >
        {{ todo.title }}
        <i @click="deleteTodo(todo.id)" class="fas fa-trash-alt"></i>
      </div>
    </div>
  </div>
</template>
<script>
import { mapGetters, mapActions } from "vuex";

export default {
  // import our getters and actions
  name: "Todos",
  methods: {
    ...mapActions(["fetchTodos", "deleteTodo", "updateTodo"]),
    onDoubleClick(currentTodo) {
      const updatedTodo = {
        id: currentTodo.id,
        title: currentTodo.title,
        completed: !currentTodo.completed,
      };
      this.updateTodo(updatedTodo);
    },
  },
  computed: {
    ...mapGetters(["allTodos"]),
  },
  created() {
    this.fetchTodos();
  },
};
</script>
<style scoped>
.todos {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 1rem;
}
.todo {
  border: 1px solid #ccc;
  background: #41b883;
  padding: 1rem;
  border-radius: 5px;
  text-align: center;
  position: relative;
  cursor: pointer;
}
i {
  position: absolute;
  bottom: 10px;
  right: 10px;
  color: #fff;
  cursor: pointer;
}
.legend {
  display: flex;
  justify-content: space-around;
  margin-bottom: 1rem;
}
.complete-box {
  padding: 2px;
  justify-content: space-around;
  width: 10px;
  height: 10px;
  background: #465462;
}
.incomplete-box {
  padding: 2px;
  width: 10px;
  height: 10px;
  background: #41b883;
}
.is-complete {
  background: #465462;
  color: #fff;
}
@media (max-width: 500px) {
  .todos {
    grid-template-columns: 1fr;
  }
}
</style>
