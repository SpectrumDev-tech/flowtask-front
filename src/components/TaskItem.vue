<script setup>
defineProps({
  task: Object
})

defineEmits(['toggle', 'delete'])

const energyConfig = {
  low: {
    label: '🔋 Низкая',
    class: 'bg-emerald-500/10 text-emerald-400 border-emerald-500/30 shadow-[0_0_15px_rgba(16,185,129,0.1)]'
  },
  medium: {
    label: '⚡️ Средняя',
    class: 'bg-amber-500/10 text-amber-400 border-amber-500/30 shadow-[0_0_15px_rgba(245,158,11,0.1)]'
  },
  high: {
    label: '🔥 Высокая',
    class: 'bg-rose-500/10 text-rose-400 border-rose-500/30 shadow-[0_0_15px_rgba(244,63,94,0.15)]'
  }
}
</script>

<template>
  <div
    class="group relative backdrop-blur-2xl bg-gradient-to-br from-white/[0.07] to-white/[0.02] border border-white/10 rounded-2xl p-5 flex items-start justify-between gap-4 transition-all duration-300 hover:border-indigo-500/40 hover:shadow-[0_8px_30px_rgb(0,0,0,0.3)] hover:-translate-y-0.5">

    <!-- Декоративный левый акцентный индиректор статуса -->
    <div
      :class="['absolute left-0 top-4 bottom-4 w-1 rounded-r-full transition-all duration-300', task.completed ? 'bg-slate-700' : 'bg-gradient-to-b from-indigo-500 to-violet-500 opacity-80 group-hover:opacity-100']">
    </div>

    <div class="flex items-start gap-4 flex-1 min-w-0 pl-2">
      <!-- Кастомный чекбокс -->
      <label class="relative flex items-center justify-center cursor-pointer mt-1">
        <input type="checkbox" :checked="task.completed" @change="$emit('toggle', task.id)" class="peer sr-only" />
        <div
          class="w-6 h-6 rounded-lg bg-slate-900/80 border border-slate-700 peer-checked:bg-indigo-600 peer-checked:border-indigo-500 transition-all duration-200 flex items-center justify-center shadow-inner">
          <svg class="w-4 h-4 text-white scale-0 peer-checked:scale-100 transition-transform duration-200" fill="none"
            viewBox="0 0 24 24" stroke="currentColor" stroke-width="3">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
          </svg>
        </div>
      </label>

      <div class="min-w-0 flex-1">
        <h3
          :class="['font-semibold text-slate-100 transition-colors text-base', task.completed ? 'line-through text-slate-500' : '']">
          {{ task.title }}
        </h3>
        <p v-if="task.description" class="text-sm text-slate-400 mt-1.5 leading-relaxed break-words">
          {{ task.description }}
        </p>

        <div class="flex items-center gap-2.5 mt-4 flex-wrap">
          <span
            :class="['text-xs px-3 py-1 rounded-lg border font-medium flex items-center gap-1.5', energyConfig[task.energy]?.class]">
            {{ energyConfig[task.energy]?.label }}
          </span>
          <span
            class="text-xs px-3 py-1 rounded-lg bg-slate-900/60 text-slate-300 border border-slate-800/80 font-medium">
            #{{ task.category }}
          </span>
        </div>
      </div>
    </div>

    <!-- Кнопка удаления с плавной анимацией появления -->
    <button @click="$emit('delete', task.id)"
      class="text-slate-500 hover:text-rose-400 p-2.5 rounded-xl hover:bg-rose-500/10 transition-all duration-200 opacity-60 group-hover:opacity-100"
      title="Удалить задачу">
      <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"
        stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round"
          d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
      </svg>
    </button>
  </div>
</template>
