<template>
  <div class="w-full max-w-md mx-auto bg-white rounded-lg shadow-lg p-6">
    <h1 class="text-2xl font-bold mb-4 text-center">My To-Do List</h1>
    <form @submit.prevent="addTodo" class="flex mb-4">
      <input
        v-model="newTodo"
        type="text"
        placeholder="Add a new task"
        class="flex-grow p-2 border rounded-l-md focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        type="submit"
        class="bg-blue-500 text-white p-2 rounded-r-md hover:bg-blue-600"
      >
        Add
      </button>
    </form>
    <ul>
      <li
        v-for="todo in todos"
        :key="todo.id"
        class="flex items-center justify-between p-2 border-b"
      >
        <span :class="{ 'line-through text-gray-500': todo.completed }">
          {{ todo.text }}
        </span>
        <div class="flex items-center">
          <button
            @click="toggleTodo(todo)"
            class="text-sm px-2 py-1 rounded mr-2"
            :class="todo.completed ? 'bg-yellow-400 hover:bg-yellow-500' : 'bg-green-500 text-white hover:bg-green-600'"
          >
            {{ todo.completed ? 'Undo' : 'Complete' }}
          </button>
          <button
            @click="removeTodo(todo.id)"
            class="bg-red-500 text-white text-sm px-2 py-1 rounded hover:bg-red-600"
          >
            Remove
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

const newTodo = ref('')
const todos = ref<Todo[]>([])

const API_URL = 'http://localhost:3000/api/todos'

async function fetchTodos() {
  const response = await fetch(API_URL)
  todos.value = await response.json()
}

async function addTodo() {
  if (newTodo.value.trim() === '') return
  const response = await fetch(API_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ text: newTodo.value }),
  })
  const newTodoItem = await response.json()
  todos.value.push(newTodoItem)
  newTodo.value = ''
}

async function toggleTodo(todo: Todo) {
  const response = await fetch(`${API_URL}/${todo.id}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ completed: !todo.completed }),
  })
  const updatedTodo = await response.json()
  const index = todos.value.findIndex(t => t.id === todo.id)
  if (index !== -1) {
    todos.value[index] = updatedTodo
  }
}

async function removeTodo(id: number) {
  await fetch(`${API_URL}/${id}`, {
    method: 'DELETE',
  })
  todos.value = todos.value.filter(t => t.id !== id)
}

onMounted(fetchTodos)
</script>
