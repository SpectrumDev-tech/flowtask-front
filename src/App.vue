<script setup>
import { ref, onMounted } from 'vue'
import TaskForm from './components/TaskForm.vue'
import FilterBar from './components/FilterBar.vue'
import TaskList from './components/TaskList.vue'

const tasks = ref([])
const currentFilter = ref('all')
const API_URL = 'http://localhost:8080/api/tasks'

async function fetchTasks() {
  try {
    const url = currentFilter.value === 'all'
      ? API_URL
      : `${API_URL}?energy=${currentFilter.value}`

    const res = await fetch(url)
    tasks.value = await res.json()
  } catch (err) {
    console.error('Ошибка загрузки задач:', err)
  }
}

async function toggleTask(id) {
  try {
    await fetch(`${API_URL}/${id}/toggle`, { method: 'PATCH' })
    await fetchTasks()
  } catch (err) {
    console.error('Ошибка обновления:', err)
  }
}

async function deleteTask(id) {
  try {
    await fetch(`${API_URL}/${id}`, { method: 'DELETE' })
    await fetchTasks()
  } catch (err) {
    console.error('Ошибка удаления:', err)
  }
}

function handleFilterChange(filter) {
  currentFilter.value = filter
  fetchTasks()
}

onMounted(fetchTasks)
</script>

<template>
  <div class="max-w-3xl mx-auto px-4 py-12">
    <!-- Шапка -->
    <header class="text-center mb-10">
      <h1
        class="text-4xl font-extrabold tracking-tight bg-gradient-to-r from-indigo-400 via-purple-400 to-pink-400 bg-clip-text text-transparent">
        FlowTask
      </h1>
      <p class="text-slate-400 mt-2 text-sm">Умный трекер задач с учетом вашей энергии и контекста</p>
    </header>

    <!-- Форма создания -->
    <TaskForm @task-created="fetchTasks" />

    <!-- Панель фильтров -->
    <FilterBar :currentFilter="currentFilter" @update:filter="handleFilterChange" />

    <!-- Список задач -->
    <TaskList :tasks="tasks" @toggle="toggleTask" @delete="deleteTask" />
  </div>
</template>
