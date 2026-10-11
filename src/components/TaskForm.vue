<script setup>
import { ref, onMounted, computed } from 'vue'

const emit = defineEmits(['task-created'])

const title = ref('')
const description = ref('')
const energy = ref('medium')
const category = ref('code')
const loading = ref(false)

// Категории
const categories = ref([])
const showAllCategories = ref(false)
const newCategoryName = ref('')
const showAddCategoryInput = ref(false)
const errorMessage = ref('')

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

// Загрузка категорий с бэкенда
async function fetchCategories() {
  try {
    const res = await fetch('http://localhost:8080/api/categories')
    const data = await res.json()
    categories.value = data
  } catch (err) {
    console.error('Ошибка загрузки категорий:', err)
  }
}

// Отображаемые категории (в свернутом виде показываем первые 4)
const visibleCategories = computed(() => {
  if (showAllCategories.value || categories.value.length <= 4) {
    return categories.value
  }
  return categories.value.slice(0, 4)
})

// Создание новой категории
async function createCategory() {
  if (!newCategoryName.value.trim()) return
  errorMessage.value = ''

  if (categories.value.length >= 10) {
    errorMessage.value = 'Достигнут лимит: максимум 10 категорий'
    return
  }

  try {
    const res = await fetch('http://localhost:8080/api/categories', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: newCategoryName.value.trim().toLowerCase() })
    })

    if (res.ok) {
      const newCat = await res.json()
      categories.value.push(newCat)
      category.value = newCat.name
      newCategoryName.value = ''
      showAddCategoryInput.value = false
    } else {
      const errData = await res.json()
      errorMessage.value = errData.error || 'Не удалось создать категорию'
    }
  } catch (err) {
    errorMessage.value = 'Ошибка сети'
  }
}

// Перемещение категории влево/вправо
async function moveCategory(index, direction) {
  const currentCat = visibleCategories.value[index]
  const globalIndex = categories.value.findIndex(c => c.id === currentCat.id)
  const targetGlobalIndex = direction === 'left' ? globalIndex - 1 : globalIndex + 1

  if (targetGlobalIndex < 0 || targetGlobalIndex >= categories.value.length) return

  // Меняем местами в глобальном массиве
  const temp = categories.value[globalIndex]
  categories.value[globalIndex] = categories.value[targetGlobalIndex]
  categories.value[targetGlobalIndex] = temp

  // Отправляем новый порядок на бэкенд
  const ids = categories.value.map(c => c.id)
  try {
    await fetch('http://localhost:8080/api/categories/reorder', {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ ids })
    })
  } catch (err) {
    console.error('Ошибка сохранения порядка категорий:', err)
  }
}

// Создание задачи
async function handleSubmit() {
  if (!title.value.trim()) return

  loading.value = true
  try {
    const res = await fetch('http://localhost:8080/api/tasks', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: title.value,
        description: description.value,
        energy: energy.value,
        category: category.value
      })
    })

    if (res.ok) {
      title.value = ''
      description.value = ''
      energy.value = 'medium'
      emit('task-created')
    }
  } catch (err) {
    console.error('Ошибка создания задачи:', err)
  } finally {
    loading.value = false
  }
}

onMounted(fetchCategories)
</script>

<template>
  <div
    class="backdrop-blur-2xl bg-gradient-to-br from-white/[0.07] to-white/[0.02] border border-white/10 rounded-2xl p-6 shadow-2xl mb-8">
    <h2 class="text-lg font-semibold mb-5 text-slate-100 tracking-tight">Новая задача</h2>

    <form @submit.prevent="handleSubmit" class="space-y-5">
      <!-- Название и описание -->
      <div class="space-y-3">
        <input v-model="title" type="text" placeholder="Что нужно сделать?"
          class="w-full px-4 py-3.5 rounded-xl bg-slate-900/60 border border-slate-800 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-indigo-500 transition text-sm shadow-inner" />
        <textarea v-model="description" placeholder="Описание или детали (необязательно)..." rows="2"
          class="w-full px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-800 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-indigo-500 transition resize-none text-sm shadow-inner"></textarea>
      </div>

      <!-- Секция: Уровень энергии -->
      <div>
        <label class="block text-xs font-semibold uppercase tracking-wider text-slate-400 mb-2.5">Требуемый уровень
          энергии</label>
        <div class="grid grid-cols-3 gap-3">
          <button type="button" v-for="(cfg, key) in energyConfig" :key="key" @click="energy = key" :class="[
            'px-4 py-2.5 rounded-xl text-xs font-medium border transition-all duration-200 text-center flex items-center justify-center gap-1.5',
            energy === key ? cfg.class + ' ring-2 ring-indigo-500/50 scale-[1.02]' : 'bg-slate-900/40 text-slate-400 border-slate-800 hover:border-slate-700 hover:text-slate-300'
          ]">
            {{ cfg.label }}
          </button>
        </div>
      </div>

      <!-- Секция: Категории с возможностью перемещения и развертывания -->
      <div>
        <div class="flex items-center justify-between mb-2.5">
          <label class="text-xs font-semibold uppercase tracking-wider text-slate-400">Категория ({{ categories.length
          }}/10)</label>

          <button v-if="categories.length > 4" type="button" @click="showAllCategories = !showAllCategories"
            class="text-xs text-indigo-400 hover:text-indigo-300 transition flex items-center gap-1 font-medium">
            <span>{{ showAllCategories ? 'Свернуть' : `Ещё (${categories.length - 4})` }}</span>
            <svg xmlns="http://www.w3.org/2000/svg"
              :class="['w-3.5 h-3.5 transition-transform duration-300', showAllCategories ? 'rotate-180' : '']"
              fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
            </svg>
          </button>
        </div>

        <!-- Список категорий -->
        <div class="flex flex-wrap gap-2 items-center transition-all duration-300 ease-in-out">
          <div v-for="(cat, index) in visibleCategories" :key="cat.id" :class="[
            'group/cat relative flex items-center gap-1 px-3.5 py-2 rounded-xl text-xs font-medium border transition-all duration-200',
            category === cat.name ? 'bg-indigo-600 text-white border-indigo-500 ring-2 ring-indigo-500/50 shadow-lg shadow-indigo-600/30' : 'bg-slate-900/40 text-slate-300 border-slate-800 hover:border-slate-700'
          ]">
            <!-- Кнопка сдв��га влево -->
            <button v-if="index > 0" type="button" @click.stop="moveCategory(index, 'left')"
              class="opacity-0 group-hover/cat:opacity-100 hover:text-white text-slate-400 transition-opacity pr-0.5 text-[10px]"
              title="Сдвинуть влево">
              ◀
            </button>

            <span @click="category = cat.name" class="cursor-pointer">#{{ cat.name }}</span>

            <!-- Кнопка сдвига вправо -->
            <button v-if="index < visibleCategories.length - 1" type="button" @click.stop="moveCategory(index, 'right')"
              class="opacity-0 group-hover/cat:opacity-100 hover:text-white text-slate-400 transition-opacity pl-0.5 text-[10px]"
              title="Сдвинуть вправо">
              ▶
            </button>
          </div>

          <!-- Кнопка добавления новой категории -->
          <button v-if="!showAddCategoryInput && categories.length < 10" type="button"
            @click="showAddCategoryInput = true"
            class="px-3.5 py-2 rounded-xl text-xs font-medium border border-dashed border-slate-700 text-slate-400 hover:text-slate-200 hover:border-slate-500 transition flex items-center gap-1">
            <span>+ Новая</span>
          </button>
        </div>

        <!-- Инпут для создания категории -->
        <div v-if="showAddCategoryInput" class="mt-3 flex gap-2">
          <input v-model="newCategoryName" type="text" placeholder="Название категории..." maxlength="15"
            @keyup.enter="createCategory"
            class="px-3.5 py-2 rounded-xl bg-slate-900/80 border border-indigo-500/50 text-xs text-slate-100 focus:outline-none flex-1 shadow-inner" />
          <button type="button" @click="createCategory"
            class="px-4 py-2 bg-indigo-600 hover:bg-indigo-500 text-white text-xs font-medium rounded-xl transition shadow-md shadow-indigo-600/30">
            Добавить
          </button>
          <button type="button" @click="showAddCategoryInput = false; errorMessage = ''"
            class="px-3.5 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs rounded-xl transition">
            Отмена
          </button>
        </div>

        <!-- Ошибка лимита / валидации -->
        <p v-if="errorMessage" class="text-xs text-rose-400 mt-2 font-medium">{{ errorMessage }}</p>
      </div>

      <!-- Кнопка отправки -->
      <div class="pt-2 flex justify-end">
        <button type="submit" :disabled="loading"
          class="w-full sm:w-auto px-6 py-3 rounded-xl bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 text-white font-medium text-sm shadow-lg shadow-indigo-500/25 transition-all duration-200 active:scale-95 disabled:opacity-50">
          Создать задачу
        </button>
      </div>
    </form>
  </div>
</template>
