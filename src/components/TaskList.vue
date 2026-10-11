<script setup>
import TaskItem from './TaskItem.vue'

defineProps({
  tasks: Array
})

defineEmits(['toggle', 'delete'])
</script>

<template>
  <div class="space-y-3">
    <div v-if="tasks.length === 0" class="text-center py-12 text-slate-500 text-sm">
      Задач пока нет. Отдохните или добавьте новую! ☕️
    </div>

    <TaskItem v-for="task in tasks" :key="task.id" :task="task" @toggle="$emit('toggle', $event)"
      @delete="$emit('delete', $event)" />
  </div>
</template>
