<script setup>
defineProps({
  currentFilter: String
})
defineEmits(['update:filter'])
</script>

<template>
  <div class="flex gap-2 mb-6 overflow-x-auto pb-2">
    <button @click="$emit('update:filter', 'all')"
      :class="['px-4 py-2 rounded-xl text-sm font-medium transition whitespace-nowrap', currentFilter === 'all' ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-600/30' : 'bg-white/5 text-slate-400 hover:bg-white/10']">
      Все задачи
    </button>
    <button @click="$emit('update:filter', 'low')"
      :class="['px-4 py-2 rounded-xl text-sm font-medium transition whitespace-nowrap', currentFilter === 'low' ? 'bg-emerald-600 text-white shadow-lg shadow-emerald-600/30' : 'bg-white/5 text-slate-400 hover:bg-white/10']">
      🔋 Низкая энергия
    </button>
    <button @click="$emit('update:filter', 'medium')"
      :class="['px-4 py-2 rounded-xl text-sm font-medium transition whitespace-nowrap', currentFilter === 'medium' ? 'bg-amber-600 text-white shadow-lg shadow-amber-600/30' : 'bg-white/5 text-slate-400 hover:bg-white/10']">
      ⚡️ Средняя энергия
    </button>
    <button @click="$emit('update:filter', 'high')"
      :class="['px-4 py-2 rounded-xl text-sm font-medium transition whitespace-nowrap', currentFilter === 'high' ? 'bg-rose-600 text-white shadow-lg shadow-rose-600/30' : 'bg-white/5 text-slate-400 hover:bg-white/10']">
      🔥 Высокая энергия
    </button>
  </div>
</template>
