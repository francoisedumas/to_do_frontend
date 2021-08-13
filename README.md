# RoR API
## Introduction to Vuex frontend using RoR API


## Starting with basic models and controllers
 - create the app
 - update App.vue
 - create the Todos.vue component
 - use Axios for API connexion
 - create the AddTodo.vue and FilterTodos.vue components
 - import Fontawesome icons

### Vue create

`vue create to_do_front`
Select the next option
`Manually select features`
Then in the choices
◉ Choose Vue version
◉ Babel
◯ TypeScript
◯ Progressive Web App (PWA) Support
◯ Router
◉ Vuex
◯ CSS Pre-processors
◉ Linter / Formatter
❯◉ Unit Testing
◯ E2E Testing
Then select
`3.x`
Then
`ESLint + Prettier`
Then
◉ Lint on save
❯◉ Lint and fix on commit
Then for testing framework
`Jest`
Then
`In dedicated config files`

### Vue App.vue

Let's update the src/App.vue file
Comments: all the futur components are already added
```javascript
<template>
  <div id="app">
    <div class="container">
      <h1>To do list manager</h1>
      <h6>Powered by: Vue 3 | Vuex 4 | Axios | RoR 6 | SQlite 3</h6>
      <AddTodo />
      <FilterTodos />
      <Todos />
    </div>
  </div>
</template>

<script>
import Todos from "./components/Todos.vue";
import AddTodo from "./components/AddTodo.vue";
import FilterTodos from "./components/FilterTodos.vue";

export default {
  name: "App",
  components: {
    Todos,
    AddTodo,
    FilterTodos,
  },
};
</script>

<style>
body {
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  line-height: 1.6;
  background: #e8f7f0;
}
.container {
  max-width: 1100px;
  margin: auto;
  overflow: auto;
  padding: 0 2em;
}
</style>
```

### Todos.vue first component
Now let's do the first component
In the src/components folder add your Todos.vue file
Comments: the `mapGetters` and `mapActions` will be done right after
```javascript
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
```
### Axios and store (mapGetters, mapActions...)

Run `npm install axios in your terminal`

In the file src/store/index.js update as below
```javascript
import { createStore } from "vuex";
import todos from "@/store/modules/todos";

export default createStore({
  modules: {
    todos
  },
});
```

Then create a folder modules in the store folder and inside add a todos.js file
Here we create all the interaction with the API
```javascript
import axios from "axios";
window.axios = axios;
const api_url = "http://localhost:3000/api/v1/todos";

const state = {
  todos: [],
};

const getters = {
  allTodos: (state) => state.todos,
};

/*
  'fetchTodos',
  'deleteTodo',
  'updateTodo'
*/

const actions = {
  async fetchTodos({ commit }) {
    const response = await axios.get(api_url);
    commit("setTodos", response.data);
  },
  async addTodo({ commit }, title) {
    const response = await axios.post(api_url,
      {
        todo: {
          title: title,
          completed: false,
        }
      });
    commit("newTodo", response.data);
  },
  async deleteTodo({ commit }, id) {
    await axios.delete(api_url + `/${id}`);
    commit("removeTodo", id);
  },
  async filterTodos({ commit }, event) {
    const limit = parseInt(event.target.options[event.target.options.selectedIndex].innerText);
    const response = await axios.get(api_url + `?_limit=${limit}`);
    console.log(response.data);
    commit("setTodos", response.data);
  },
  async updateTodo({ commit }, updatedTodo) {
    const response = await axios.put(
      api_url + `/${updatedTodo.id}`,
      updatedTodo
    );
    commit("setUpdatedTodo", response.data);
  }
};

const mutations = {
  setTodos: (state, todos) => (state.todos = todos),
  newTodo: (state, todo) => (state.todos.unshift(todo)),
  removeTodo: (state, id) =>
    (state.todos = state.todos.filter((todo) => todo.id !== id)),
  setUpdatedTodo: (state, updatedTodo) => {
    const index = state.todos.findIndex((todo) => todo.id === updatedTodo.id);
    if (index !== -1) {
      state.todos.splice(index, 1, updatedTodo);
    }
  },
};

export default {
  state,
  getters,
  actions,
  mutations,
};
```

### AddTodo.vue component
In the src/components folder add your AddTodo.vue file
```javascript
<template>
  <div>
    <h3>Add Todo</h3>
    <div id="add">
      <form @submit="onSubmit">
        <input type="text" v-model="title" placeholder="Add Todo..." />
        <input type="submit" value="Submit" />
      </form>
    </div>
  </div>
</template>

<script>
import { mapActions } from "vuex";

export default {
  name: "addTodo",
  data() {
    return {
      title: "",
    };
  },
  methods: {
    ...mapActions(["addTodo"]),
    onSubmit(event) {
      event.preventDefault();
      this.addTodo(this.title);
      this.title = "";
    },
  },
};
</script>

<style scoped>
form {
  display: flex;
}
input[type="text"] {
  flex: 10;
  padding: 10px;
  border: 1px solid #ccc;
  outline: 0;
}
input[type="text"]:hover,
input[type="text"]:focus {
  border: 1px solid #41b883;
}
input[type="submit"] {
  flex: 2;
  background: #41b883;
  color: #fff;
  border: 1px #41b883 solid;
  cursor: pointer;
}
input[type="submit"]:hover,
input[type="submit"]:focus {
  background: #318b63;
}
</style>
```

### FilterTodos.vue component
In the src/components folder add your FilterTodos.vue file
```javascript
<template>
  <div>
    Filter Todos:
    <select @change="filterTodos($event)">
      <option value="100">100</option>
      <option value="50">50</option>
      <option value="10">10</option>
      <option value="5">5</option>
    </select>
  </div>
</template>

<script>
import { mapActions } from "vuex";
export default {
  name: "FilterTodos",
  methods: mapActions(["filterTodos"]),
}
</script>

<style scoped>
select {
  margin-top: 20px;
  padding: 6px;
  border: #41b883;
}
</style>
```

### Fontawesome icons
in the head of public/index.html file add your personnal Fontawesome Kit's Code
```html
<script src="https://kit.fontawesome.com/02bad37339.js" crossorigin="anonymous"></script>
```
