<template>
  <div class="todo-app">
    <h1>Todo List dengan Axios & JSON Server (CRUD lengkap)</h1>

    <form @submit.prevent="addTodo">
      <input v-model="newTodo" placeholder="Tambah todo baru...." required />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <p v-if="loading">Memuat data...</p>
    <p v-if="error" style="color: red;">{{ error }}</p>

    <ul v-if="todos.length">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />

        <template v-if="editedId !== todo.id">
          <span :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
            {{ todo.title }}
          </span>
          <button @click="startEdit(todo)">Edit</button>
        </template>

        <template v-else>
          <input v-model="editedTitle" />
          <button @click="updateTodo(todo)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </template>

        <button @click="deleteTodo(todo.id)">Hapus</button>
      </li>
    </ul>

    <p v-else>Tidak ada todo.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const todos = ref([])
const newTodo = ref('')
const editedId = ref(null)
const editedTitle = ref('')
const loading = ref(false)
const error = ref(null)

const fetchTodos = async () => {
  try {
    loading.value = true
    const res = await axios.get('http://localhost:3000/todos')
    todos.value = res.data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const addTodo = async () => {
  if (!newTodo.value) return
  try {
    loading.value = true
    await axios.post('http://localhost:3000/todos', {
      title: newTodo.value,
      completed: false
    })
    newTodo.value = ''
    fetchTodos()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const toggleCompleted = async (todo) => {
  try {
    await axios.patch(`http://localhost:3000/todos/${todo.id}`, {
      completed: !todo.completed
    })
    fetchTodos()
  } catch (err) {
    error.value = err.message
  }
}

const deleteTodo = async (id) => {
  try {
    await axios.delete(`http://localhost:3000/todos/${id}`)
    fetchTodos()
  } catch (err) {
    error.value = err.message
  }
}

const startEdit = (todo) => {
  editedId.value = todo.id
  editedTitle.value = todo.title
}

const cancelEdit = () => {
  editedId.value = null
  editedTitle.value = ''
}

const updateTodo = async (todo) => {
  try {
    await axios.patch(`http://localhost:3000/todos/${todo.id}`, {
      title: editedTitle.value
    })
    cancelEdit()
    fetchTodos()
  } catch (err) {
    error.value = err.message
  }
}

onMounted(fetchTodos)
</script>

<style scoped>
.todo-app {
  max-width: 550px;
  margin: 60px auto;
  padding: 40px;
  background: linear-gradient(145deg, #f0f4ff, #dbe8ff);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.15);
  border-radius: 20px;
  font-family: 'Segoe UI', sans-serif;
  transition: 0.3s;
}

h1 {
  text-align: center;
  margin-bottom: 25px;
  color: #2c3e50;
  font-size: 24px;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  justify-content: center;
}

input[type="text"],
input[type="checkbox"],
input {
  padding: 10px 14px;
  border: 2px solid #ddd;
  border-radius: 10px;
  font-size: 15px;
  outline: none;
  transition: border 0.2s, box-shadow 0.2s;
}

input:focus {
  border-color: #3498db;
  box-shadow: 0 0 8px rgba(52, 152, 219, 0.4);
}

button {
  padding: 10px 16px;
  background: linear-gradient(to right, #3498db, #6dd5fa);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
}

button:hover {
  background: linear-gradient(to right, #2980b9, #48c6ef);
}

button:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
}

ul {
  list-style: none;
  padding-left: 0;
}

li {
  display: flex;
  align-items: center;
  margin: 12px 0;
  background: #ffffff;
  padding: 12px 14px;
  border-radius: 12px;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
  transition: 0.2s;
}

li:hover {
  transform: translateY(-2px);
}

li span {
  flex-grow: 1;
  margin-left: 10px;
  color: #2c3e50;
}

li input[type="text"] {
  flex-grow: 1;
  padding: 8px 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
}

p {
  text-align: center;
  color: #7f8c8d;
}
</style>
