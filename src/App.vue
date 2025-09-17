<template>
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-50 p-4">
    <!-- Modal: pedir importe inicial si no hay -->
    <div v-if="initialRequired" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div class="bg-white rounded-2xl shadow-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-semibold mb-3">Ingresa el importe inicial</h2>
        <p class="text-sm text-gray-600 mb-4">Este será el saldo base sobre el que se aplicarán ingresos y gastos.</p>
        <input
          v-model.number="initialInput"
          @keyup.enter="saveInitial"
          ref="initialInputRef"
          type="number"
          step="0.01"
          placeholder="Ej: 1000"
          class="w-full px-3 py-2 border rounded mb-4 focus:outline-none focus:ring"
        />
        <div class="flex justify-end gap-2">
          <button @click="initialInput = 0" class="px-3 py-2 rounded bg-gray-200">Poner 0</button>
          <button @click="saveInitial" class="px-4 py-2 rounded bg-blue-600 text-white">Guardar</button>
        </div>
      </div>
    </div>

    <!-- Contenedor principal -->
    <div class="w-full max-w-3xl mx-auto">
      <div class="bg-white rounded-2xl shadow p-6 flex flex-col gap-6">
        <header class="text-center">
          <h1 class="text-2xl font-bold">App de Finanzas Personales</h1>
          <p class="text-sm text-gray-600 mt-1">Saldo base: <span class="font-medium">{{ formatCurrency(initialAmount) }}</span></p>
          <p class="text-lg mt-2">Balance actual: <span :class="balanceClass" class="font-semibold">{{ formatCurrency(currentBalance) }}</span></p>
        </header>

        <!-- Formulario -->
        <form @submit.prevent="onSave" class="flex flex-col md:flex-row gap-3 items-start md:items-end">
          <div class="flex-1 flex flex-col gap-2">
            <label class="text-sm">Concepto</label>
            <input ref="conceptRef" v-model="form.concept" class="px-3 py-2 border rounded w-full focus:outline-none focus:ring" placeholder="Ej: Pago luz" />

            <div class="grid grid-cols-2 gap-2 mt-2">
              <div>
                <label class="text-sm">Importe</label>
                <input v-model.number="form.amount" type="number" step="0.01" class="px-3 py-2 border rounded w-full focus:outline-none focus:ring" placeholder="Ej: 123.45" />
              </div>

              <div>
                <label class="text-sm">Tipo</label>
                <select v-model="form.type" class="px-3 py-2 border rounded w-full focus:outline-none focus:ring">
                  <option value="gasto">Gasto</option>
                  <option value="ingreso">Ingreso</option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex flex-col gap-2 md:w-56">
            <button type="button" @click="clearForm" class="px-4 py-2 rounded border">Limpiar</button>
            <button type="submit" class="px-4 py-2 rounded bg-green-600 text-white">{{ editingId === null ? 'Guardar' : 'Actualizar' }}</button>
          </div>
        </form>

        <!-- Lista de registros -->
        <div>
          <h2 class="font-semibold mb-2">Registros</h2>
          <div v-if="entries.length === 0" class="text-sm text-gray-500">No hay registros aún.</div>

          <ul class="divide-y">
            <li v-for="entry in entries" :key="entry.id" class="py-3 flex flex-col md:flex-row md:items-center md:justify-between gap-2">
              <div>
                <div class="flex items-center gap-3">
                  <span class="text-sm font-medium">{{ entry.concept }}</span>
                  <span class="text-xs text-gray-500">• {{ formatDate(entry.date) }}</span>
                  <span class="ml-3 px-2 py-0.5 text-xs rounded-full" :class="entry.type === 'ingreso' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">{{ entry.type }}</span>
                </div>
                <div class="text-sm text-gray-600 mt-1">{{ formatCurrency(entry.amount) }}</div>
              </div>

              <div class="flex items-center gap-2">
                <button @click="startEdit(entry)" class="px-3 py-1 rounded border">Editar</button>
                <button @click="removeEntry(entry.id)" class="px-3 py-1 rounded border text-red-600">Eliminar</button>
              </div>
            </li>
          </ul>
        </div>

      </div>
    </div>

    <!-- Footer fijo -->
    <footer class="fixed bottom-4 left-0 right-0 flex justify-center z-40">
      <div class="bg-white/90 backdrop-blur rounded-full px-4 py-2 shadow flex items-center gap-4">
        <a href="https://github.com/yourusername" target="_blank" class="flex items-center gap-2 hover:underline">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="currentColor"><path d="M12 .5C5.73.5.5 5.73.5 12c0 5.08 3.29 9.39 7.86 10.91.57.1.78-.25.78-.55 0-.27-.01-1.15-.02-2.08-3.2.7-3.88-1.39-3.88-1.39-.52-1.33-1.27-1.68-1.27-1.68-1.04-.71.08-.7.08-.7 1.15.08 1.75 1.18 1.75 1.18 1.02 1.75 2.67 1.25 3.32.96.1-.75.4-1.25.72-1.54-2.55-.29-5.24-1.28-5.24-5.7 0-1.26.45-2.29 1.18-3.1-.12-.29-.51-1.47.11-3.06 0 0 .97-.31 3.18 1.18.92-.26 1.91-.39 2.89-.39.98 0 1.97.13 2.89.39 2.2-1.49 3.17-1.18 3.17-1.18.62 1.59.23 2.77.11 3.06.74.81 1.18 1.84 1.18 3.1 0 4.43-2.7 5.4-5.27 5.68.41.35.77 1.04.77 2.1 0 1.52-.01 2.75-.01 3.12 0 .3.2.66.79.55C20.71 21.39 24 17.08 24 12c0-6.27-5.23-11.5-12-11.5z"/></svg>
          <span class="text-sm">GitHub</span>
        </a>

        <a href="https://www.linkedin.com/in/yourprofile" target="_blank" class="flex items-center gap-2 hover:underline">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="currentColor"><path d="M4.98 3.5C4.98 4.88 3.88 6 2.5 6S0 4.88 0 3.5 1.12 1 2.5 1 4.98 2.12 4.98 3.5zM.5 8h4V24h-4zM9 8h3.8v2.2h.1c.5-.9 1.7-1.9 3.5-1.9 3.8 0 4.5 2.5 4.5 5.8V24H19v-7.3c0-1.7 0-3.8-2.3-3.8-2.3 0-2.7 1.8-2.7 3.6V24H9z"/></svg>
          <span class="text-sm">LinkedIn</span>
        </a>

        <a href="mailto:youremail@hotmail.com" class="flex items-center gap-2 hover:underline">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="currentColor"><path d="M12 12.713L.8 4.5V20h22.4V4.5L12 12.713zM12 2 0 9l12 7 12-7L12 2z"/></svg>
          <span class="text-sm">Hotmail</span>
        </a>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch, nextTick } from 'vue'

const STORAGE_KEY = 'finanzas_entries_v1'
const STORAGE_INITIAL_KEY = 'finanzas_initial_v1'

// Estado
const entries = ref([])
const initialAmount = ref(0)
const initialInput = ref(null)
const initialRequired = ref(false)

const form = reactive({ concept: '', amount: null, type: 'gasto' })
const editingId = ref(null)

// Refs para foco
const conceptRef = ref(null)
const initialInputRef = ref(null)

// Computed
const currentBalance = computed(() => {
  const sum = entries.value.reduce((acc, e) => {
    return acc + (e.type === 'ingreso' ? Number(e.amount) : -Number(e.amount))
  }, 0)
  return Number(initialAmount.value) + sum
})

const balanceClass = computed(() => currentBalance.value < 0 ? 'text-red-600' : 'text-green-600')

// Helpers
function formatCurrency(v) {
  const num = Number(v) || 0
  return num.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })
}

function formatDate(ts) {
  const d = new Date(ts)
  return d.toLocaleString()
}

// CRUD localStorage
function load() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    entries.value = raw ? JSON.parse(raw) : []
  } catch (e) {
    entries.value = []
  }
  const initRaw = localStorage.getItem(STORAGE_INITIAL_KEY)
  if (initRaw === null) {
    initialRequired.value = true
  } else {
    initialAmount.value = Number(initRaw) || 0
  }
}

function persist() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(entries.value))
}

function persistInitial() {
  localStorage.setItem(STORAGE_INITIAL_KEY, String(initialAmount.value))
}

// Actions
function saveInitial() {
  const v = Number(initialInput.value)
  if (isNaN(v)) return
  initialAmount.value = v
  persistInitial()
  initialRequired.value = false
  // focus concepto
  nextTick(() => conceptRef.value?.focus())
}

function clearForm() {
  form.concept = ''
  form.amount = null
  form.type = 'gasto'
  editingId.value = null
  nextTick(() => conceptRef.value?.focus())
}

function onSave() {
  if (!form.concept || !form.amount || Number(form.amount) === 0) return
  if (editingId.value === null) {
    const newEntry = {
      id: Date.now(),
      concept: form.concept.trim(),
      amount: Number(form.amount),
      type: form.type,
      date: new Date().toISOString()
    }
    entries.value.unshift(newEntry)
  } else {
    const idx = entries.value.findIndex(e => e.id === editingId.value)
    if (idx !== -1) {
      entries.value[idx].concept = form.concept.trim()
      entries.value[idx].amount = Number(form.amount)
      entries.value[idx].type = form.type
      entries.value[idx].date = new Date().toISOString()
    }
    editingId.value = null
  }
  persist()
  clearForm()
}

function startEdit(entry) {
  form.concept = entry.concept
  form.amount = entry.amount
  form.type = entry.type
  editingId.value = entry.id
  nextTick(() => conceptRef.value?.focus())
}

function removeEntry(id) {
  entries.value = entries.value.filter(e => e.id !== id)
  persist()
}

// Init
onMounted(() => {
  load()
  nextTick(() => {
    if (initialRequired.value) {
      // focus input modal
      initialInput.value = null
      nextTick(() => initialInputRef.value?.focus())
    } else {
      conceptRef.value?.focus()
    }
  })
})

// Auto persist when entries change
watch(entries, persist, { deep: true })
</script>

<style>
/* Component-level styles minimal: Tailwind makes most styling */
</style>