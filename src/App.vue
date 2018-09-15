<template>
  <div>
    <input
      type="text"
      placeholder="Todo text"
      v-model="todoText"
    />
    <button @click="createTodo">Create</button>
    &nbsp;
    <button @click="deleteDone">Delete done</button>
    <hr />
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        <input
          :id="'todocheck_' + todo.id"
          type="checkbox"
          :checked="todo.done"
          @change="toggleTodo(todo)"
        />

        &nbsp;

        <label :for="'todocheck_' + todo.id">
          <span v-bind:class="{ itsDone: todo.done }">{{todo.text}}</span>
        </label>
      </li>
    </ul>
  </div>

</template>

<script>
export default {
  name: "app",

  data() {
    return {
      token: "",
      todoText: "",
      todos: []
    };
  },

  created() {
    this.getToken().then(() => this.getTodos());
  },

  methods: {
    async getToken() {
      let response = await fetch("http://127.0.0.1:8000/token-auth/", {
        method: "POST",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ username: "admin", password: "admin123" })
      });

      let data = await response.json();
      this.token = data.token;
    },

    async getTodos() {
      let response = await fetch("http://127.0.0.1:8000/todos/", {
        method: "GET",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
          Authorization: `JWT ${this.token}`
        }
      });
      let data = await response.json();

      this.todos = data;
    },

    async createTodo() {
      const text = this.todoText.trim();

      if (text.length === 0) {
        console.log("Nop, nop nop nop...");
        return;
      }

      let response = await fetch("http://127.0.0.1:8000/todos/", {
        method: "POST",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
          Authorization: `JWT ${this.token}`
        },
        body: JSON.stringify({ text, done: false })
      });

      let data = await response.json();

      this.todoText = "";
      this.todos.push(data);
    },

    async toggleTodo(todo) {
      fetch(`http://127.0.0.1:8000/todos/${todo.id}/`, {
        method: "PATCH",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
          Authorization: `JWT ${this.token}`
        },
        body: JSON.stringify({ done: !todo.done })
      });
    },

    async deleteDone() {
      let confirmation = window.confirm("Are you sure ?");

      if (!confirmation) {
        return;
      }

      const doneTodos = this.todos.filter(todo => todo.done);

      const promises = doneTodos.map(todo => {
        /**
         * Yeah ugly...
         * but it works and I don't want to create a custom route
         */
        fetch(`http://127.0.0.1:8000/todos/${todo.id}/`, {
          method: "DELETE",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
            Authorization: `JWT ${this.token}`
          }
        });
      });

      await Promise.all(promises);

      const notDone = this.todos.filter(todo => !todo.done);
      this.todos = notDone;
    }
  }
};
</script>

<style>
.itsDone {
  text-decoration: line-through;
}
</style>
