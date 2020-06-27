<template>
  <div>
    <!-- <input type="text" placeholder @keyup.enter="addTodo()" v-model="newTodo" /> -->
    <b-form-input
      v-model="newTodo"
      placeholder="works to do"
      @keyup.enter="addTodo()"
    ></b-form-input>
    <b-list-group
      v-for="(todo, index) in todosFiltered"
      :key="todo.id"
      class="todo-item"
    >
      <b-list-group-item
        class="d-flex justify-content-between align-items-center"
      >
        <div
          v-if="!todo.editing"
          @dblclick="editToto(todo)"
          :class="{ completed: todo.isComplete }"
        >
          <b-form-checkbox
            v-model="todo.isComplete"
            @change="completed(todo)"
            :checked="todo.isComplete"
            class="float-left"
          ></b-form-checkbox>
          {{ todo.title }}
        </div>
        <b-form-input
          v-else
          v-model="todo.title"
          placeholder="press esc to cancel"
          @keyup.enter="doneTodo(todo)"
          @blur="doneTodo(todo)"
          @keyup.esc="cancelEdit(todo)"
          v-focus
        ></b-form-input>
        <b-badge
          class="cursor-pointer"
          @click="deleteTodo(index, todo.id)"
          variant="primary"
          pill
          >X</b-badge
        >
      </b-list-group-item>
    </b-list-group>
    <div
      v-if="showButton"
      class="d-flex justify-content-between align-items-center mt-3"
    >
      <div>{{ remaining }} item(s) left</div>
      <div>
        <b-button
          variant="outline-primary"
          size="sm"
          :class="{ active: filter == 'all' }"
          @click="filter = 'all'"
          >all</b-button
        >
        <b-button
          variant="outline-primary"
          size="sm"
          :class="{ active: filter == 'active' }"
          @click="filter = 'active'"
          >active</b-button
        >
        <b-button
          variant="outline-primary"
          size="sm"
          :class="{ active: filter == 'completed' }"
          @click="filter = 'completed'"
          >completed</b-button
        >
      </div>
      <div>
        <b-button
          v-if="showClearCompletedButton"
          @click="clearCompleted"
          variant="outline-primary"
          size="sm"
          >clear completed</b-button
        >
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";

axios.defaults.baseURL = "http://127.0.0.1:8000/api";

export default {
  name: "todo",
  data() {
    return {
      newTodo: "",
      idForTodo: 100,
      beforeEditCache: "",
      filter: "all",
      todos: []
    };
  },
  mounted: function() {
    axios
      .get("/tasks")
      .then(response => {
        let todoData = response.data;
        todoData.forEach(todo => {
          todo.editing = false;
        });
        this.todos = todoData;
      })
      .catch(error => console.log(error));
  },
  computed: {
    remaining() {
      return this.todos.filter(todo => !todo.isComplete).length;
    },
    todosFiltered() {
      if (this.filter == "all") {
        return this.todos;
      } else if (this.filter == "active") {
        return this.todos.filter(todo => !todo.isComplete);
      } else if (this.filter == "completed") {
        return this.todos.filter(todo => todo.isComplete);
      }
      return this.todos;
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.isComplete).length > 0;
    },
    showButton() {
      return this.todos.length > 0;
    }
  },
  directives: {
    focus: {
      inserted: el => el.focus()
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim() == 0) {
        return;
      }
      axios
        .post("/tasks", {
          title: this.newTodo,
          isComplete: false
        })
        .then(response => {
          let todoData = response.data;
          todoData.editing = false;
          this.todos.push(todoData);
        })
        .catch(error => console.log(error));
      this.newTodo = "";
      this.idForTodo++;
    },
    deleteTodo(index, id) {
      axios
        .delete("/tasks/" + id)
        .then(response => {
          if (response.status == 200) {
            this.todos.splice(index, 1);
          }
        })
        .catch(error => console.log(error));
    },
    editToto(todo) {
      this.beforeEditCache = todo.title;
      todo.editing = true;
    },
    doneTodo(todo) {
      if (todo.title.trim() == "") {
        todo.title = this.beforeEditCache;
      }
      axios
        .put("/tasks/" + todo.id, {
          title: todo.title
        })
        .then(response => {
          todo.editing = false;
        })
        .catch(error => console.log(error));
    },
    completed(todo) {
      axios
        .put("/tasks/" + todo.id, {
          isComplete: !todo.isComplete
        })
        .then(response => console.log(response))
        .catch(error => console.log(error));
    },
    cancelEdit(todo) {
      todo.title = this.beforeEditCache;
      todo.editing = false;
    },
    clearCompleted() {
      this.todos.forEach(todo => {
        if (todo.isComplete) {
          axios
            .delete("/tasks/" + todo.id)
            .then(response => {
              if (response.status == 200) {
                this.todos = this.todos.filter(
                  todo => todo.id != response.data.id
                );
              }
            })
            .catch(error => console.log(error));
        }
      });
    }
  }
};
</script>
<style>
.cursor-pointer {
  cursor: pointer;
}
.completed {
  text-decoration: line-through;
  color: gray;
}
</style>
