<template>
  <div class="wrapper">
    <!-- Navbar dan form Todo -->
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
      <!-- ... -->
    </nav>

    <div class="container py-5">
      <div class="row d-flex justify-content-center align-items-center">
        <div class="col-10">
          <div class="card rounded">
            <div class="card-body p-5">
              <!-- Judul dan total tugas -->
              <h3 class="mb-3 fw-bold">
                To-Do List
                <span class="badge bg-primary rounded-pill ms-3 fs-6 fw-normal">{{ totalTodo }} task</span>
              </h3>

              <!-- Form Todo -->
              <form @submit.prevent="addTodo" class="row align-items-center mb-3 gy-2">
                <div class="col-sm-12 col-md-10">
                  <div class="form-floating">
                    <input type="text" class="form-control fs-4" id="floatingInput" placeholder="Todos" autocomplete="off" v-model="todo" />
                    <label for="floatingInput">Apa rencana kegiatanmu hari ini?</label>
                  </div>
                </div>
                <div class="col-sm-12 col-md-2">
                  <div class="d-grid">
                    <button type="submit" class="btn btn-primary btn-lg" :disabled="todo.length === 0">
                      <font-awesome-icon icon="circle-plus" />
                      Daftar Rencana Hari Ini
                    </button>
                  </div>
                </div>
              </form>

              <!-- Daftar Todo -->
              <ul class="list-group list-group-flush">
                <li v-for="(todo, index) in todos" :key="index" class="list-group-item">
                  <div class="row gy-2">
                    <div class="col-sm-12 col-md-auto me-auto align-self-center fs-5"
                         :class="{ 'text-decoration-line-through text-muted': todo.isDone }"
                         v-if="editedTodoIndex !== index">
                      {{ todo.activity }}
                    </div>
                    <div class="col-sm-12 col-md-auto"
                         v-else>
                      <input v-model="editedTodoText" class="form-control" @keyup.enter="saveEdit(index)" @keyup.escape="cancelEdit">
                    </div>
                    <div class="col-sm-12 col-md-auto">
                      <div class="row gx-2">
                        <div class="col-auto">
                          <button class="btn"
                                  :class="[
                                    todo.isDone ? 'btn-outline-secondary' : 'btn-outline-success',
                                  ]"
                                  @click="doneTodo(index)">
                            <font-awesome-icon icon="circle-check" />
                          </button>
                        </div>
                        <div class="col-auto">
                          <button class="btn"
                                  :class="[
                                    todo.isDone ? 'btn-outline-secondary' : 'btn-outline-danger',
                                  ]"
                                  @click="deleteTodo(index)">
                            <font-awesome-icon icon="eraser" />
                          </button>
                        </div>
                        <div class="col-auto">
                          <button class="btn btn-outline-primary"
                                  v-if="editedTodoIndex !== index"
                                  @click="editTodo(index, todo.activity)">
                            <font-awesome-icon icon="edit" />
                          </button>
                          <button class="btn btn-outline-secondary"
                                  v-else
                                  @click="cancelEdit">
                            <font-awesome-icon icon="times" />
                          </button>
                          <button class="btn btn-outline-secondary"
                                  v-if="editedTodoIndex === index"
                                  @click="saveEdit(index)">
                            <font-awesome-icon icon="check" />
                          </button>
                        </div>
                      </div>
                    </div>
                  </div>
                </li>
              </ul>

              <!-- Daftar Pengguna -->
              <select v-model="selectedUser" @change="fetchPosts" class="form-select mt-3">
                <option value="" disabled selected>Pilih pengguna</option>
                <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
              </select>

              <!-- Judul dan daftar Post -->
              <div v-if="posts.length > 0">
                <h3 class="mb-3 fw-bold">Posts</h3>
                <ul class="list-group">
                  <li v-for="(post, index) in posts" :key="index" class="list-group-item">
                    <h5 class="card-title">{{ post.title }}</h5>
                    <p class="card-text">{{ post.body }}</p>
                  </li>
                </ul>
              </div>
              <div v-else>
                <p>No posts available.</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      todo: "",
      todos: [],
      posts: [],
      editedTodoIndex: null,
      editedTodoText: "",
      users: [],
      selectedUser: null,
    };
  },
  mounted() {
    if (localStorage.getItem("todos") !== null) {
      this.todos = JSON.parse(localStorage.getItem("todos"));
    }
    this.fetchUsers();
  },
  methods: {
    addTodo() {
      if (this.todo.trim() !== "") {
        this.todos.unshift({
          activity: this.todo,
          isDone: false,
          editing: false,
        });
        this.todo = "";
        this.saveToLocalStorage();
      }
    },
    deleteTodo(indexDelete) {
      this.todos.splice(indexDelete, 1);
      this.saveToLocalStorage();
    },
    doneTodo(indexDone) {
      this.todos[indexDone].isDone = !this.todos[indexDone].isDone;
      this.saveToLocalStorage();
    },
    editTodo(index, text) {
      this.editedTodoIndex = index;
      this.editedTodoText = text;
    },
    cancelEdit() {
      this.editedTodoIndex = null;
      this.editedTodoText = "";
    },
    saveEdit(index) {
      if (this.editedTodoText.trim() !== "") {
        this.todos[index].activity = this.editedTodoText;
        this.editedTodoIndex = null;
        this.editedTodoText = "";
        this.saveToLocalStorage();
      }
    },
    saveToLocalStorage() {
      localStorage.setItem("todos", JSON.stringify(this.todos));
    },
    async fetchUsers() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        const data = await response.json();
        this.users = data;
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },
    async fetchPosts() {
      if (!this.selectedUser) return;
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${this.selectedUser}`);
        const data = await response.json();
        this.posts = data;
      } catch (error) {
        console.error('Error fetching posts:', error);
      }
    },
  },
  computed: {
    totalTodo() {
      return this.todos.length;
    },
  },
};
</script>

<style>
@import "./assets/styles.css";
</style>
