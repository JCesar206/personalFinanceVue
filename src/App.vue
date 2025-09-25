<script setup>
import { ref, onMounted, watch } from "vue";

// Estado inicial
const accountName = ref("");
const initialBalance = ref("");
const balance = ref(0);
const movements = ref([]);
const isEditing = ref(false);
const editIndex = ref(null);

// Inputs movimiento
const amount = ref("");
const category = ref("");
const description = ref("");
const type = ref("ingreso"); // ingreso | gasto

// Dark mode
const darkMode = ref(false);

// Cargar datos de localStorage
onMounted(() => {
  const stored = JSON.parse(localStorage.getItem("financeApp"));
  if (stored) {
    accountName.value = stored.accountName;
    balance.value = stored.balance;
    movements.value = stored.movements;
  }
  const theme = localStorage.getItem("darkMode");
  if (theme) darkMode.value = JSON.parse(theme);
});

// Guardar datos en localStorage
watch([accountName, balance, movements], () => {
  localStorage.setItem(
    "financeApp",
    JSON.stringify({
      accountName: accountName.value,
      balance: balance.value,
      movements: movements.value,
    })
  );
}, { deep: true });

watch(darkMode, () => {
  localStorage.setItem("darkMode", JSON.stringify(darkMode.value));
});

// Inicializar cuenta
const initAccount = () => {
  if (!accountName.value || !initialBalance.value) return;
  balance.value = parseFloat(initialBalance.value);
  initialBalance.value = "";
};

// Agregar o editar movimiento
const addMovement = () => {
  if (!amount.value || !category.value || !description.value) return;

  const signedAmount = type.value === "gasto" ? -Math.abs(parseFloat(amount.value)) : Math.abs(parseFloat(amount.value));

  const newMovement = {
    amount: signedAmount,
    category: category.value,
    description: description.value,
    type: type.value,
    date: new Date().toLocaleString(),
  };

  if (isEditing.value) {
    // revertir el saldo previo
    balance.value -= movements.value[editIndex.value].amount;
    movements.value[editIndex.value] = newMovement;
    isEditing.value = false;
    editIndex.value = null;
  } else {
    movements.value.push(newMovement);
  }

  balance.value += signedAmount;

  clearFields();
};

// Editar movimiento
const editMovement = (index) => {
  const mov = movements.value[index];
  amount.value = Math.abs(mov.amount);
  category.value = mov.category;
  description.value = mov.description;
  type.value = mov.type;
  isEditing.value = true;
  editIndex.value = index;
};

// Eliminar movimiento
const deleteMovement = (index) => {
  balance.value -= movements.value[index].amount;
  movements.value.splice(index, 1);
};

// Limpiar campos
const clearFields = () => {
  amount.value = "";
  category.value = "";
  description.value = "";
  type.value = "ingreso";
};
</script>

<template>
  <div :class="darkMode ? 'dark' : ''">
    <div class="min-h-screen flex flex-col justify-between bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
      <main class="flex flex-col items-center justify-center px-4 py-6">
        <!-- Toggle Dark Mode -->
        <button
          @click="darkMode = !darkMode"
          class="mb-6 px-4 py-2 rounded-lg bg-indigo-600 text-white hover:bg-indigo-700 cursor-pointer"
        >
          {{ darkMode ? '☀️ Modo Claro' : '🌙 Modo Oscuro' }}
        </button>

        <!-- Configuración inicial -->
        <div v-if="!balance && !movements.length" class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow w-full max-w-md text-center">
          <h2 class="text-xl font-bold mb-4">Configura tu cuenta</h2>
          <input
            v-model="accountName"
            placeholder="Nombre de la cuenta"
            class="w-full mb-3 p-2 rounded border dark:bg-gray-700"
          />
          <input
            v-model="initialBalance"
            type="number"
            placeholder="Saldo inicial"
            class="w-full mb-3 p-2 rounded border dark:bg-gray-700"
          />
          <button
            @click="initAccount"
            class="w-full py-2 bg-green-600 text-white rounded hover:bg-green-700 cursor-pointer"
          >
            Guardar cuenta
          </button>
        </div>

        <!-- App Finanzas -->
        <div v-else class="bg-white dark:bg-gray-800 p-6 rounded-2xl shadow w-full max-w-2xl">
          <h1 class="text-2xl font-bold mb-2 text-center">{{ accountName }}</h1>
          <p class="text-center mb-6 text-lg">Saldo actual: 
            <span :class="balance >= 0 ? 'text-green-600' : 'text-red-600'">
              {{ balance.toFixed(2) }}
            </span>
          </p>

          <!-- Formulario -->
          <div class="grid gap-3 mb-4">
            <input
              v-model="amount"
              type="number"
              placeholder="Cantidad"
              class="w-full p-2 rounded border dark:bg-gray-700"
            />
            <input
              v-model="category"
              placeholder="Categoría"
              class="w-full p-2 rounded border dark:bg-gray-700"
            />
            <input
              v-model="description"
              placeholder="Descripción"
              class="w-full p-2 rounded border dark:bg-gray-700"
            />

            <select
              v-model="type"
              class="w-full p-2 rounded border dark:bg-gray-700"
            >
              <option value="ingreso">Ingreso</option>
              <option value="gasto">Gasto</option>
            </select>

            <div class="flex gap-2">
              <button
                @click="addMovement"
                class="flex-1 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700 cursor-pointer"
              >
                {{ isEditing ? 'Guardar cambios' : 'Agregar' }}
              </button>
              <button
                @click="clearFields"
                class="flex-1 py-2 bg-gray-400 text-white rounded hover:bg-gray-500 cursor-pointer"
              >
                Limpiar
              </button>
            </div>
          </div>

          <!-- Lista de movimientos -->
          <h2 class="text-xl font-semibold mb-3">Movimientos</h2>
          <ul class="space-y-2 max-h-64 overflow-y-auto">
            <li
              v-for="(mov, index) in movements"
              :key="index"
              class="flex justify-between items-center bg-gray-100 dark:bg-gray-700 p-3 rounded-lg"
            >
              <div>
                <p class="font-semibold">{{ mov.description }} - {{ mov.category }} ({{ mov.type }})</p>
                <p class="text-sm text-gray-500">{{ mov.date }}</p>
              </div>
              <div class="flex items-center gap-2">
                <span :class="mov.amount >= 0 ? 'text-green-600' : 'text-red-600'">
                  {{ mov.amount }}
                </span>
                <button @click="editMovement(index)" class="px-2 py-1 bg-yellow-500 text-white rounded hover:bg-yellow-600 cursor-pointer">✏️</button>
                <button @click="deleteMovement(index)" class="px-2 py-1 bg-red-600 text-white rounded hover:bg-red-700 cursor-pointer">🗑️</button>
              </div>
            </li>
          </ul>
        </div>
      </main>

      <!-- Footer -->
      <footer class="w-full bg-gray-200 dark:bg-gray-800 text-center py-4 mt-6 font-semibold">
        <div class="flex justify-center gap-6 mb-2">
          <a href="https://jcesar206.github.io/myPersonalBlog/" target="_blank" class="hover:underline">Home Page</a>
          <a href="https://github.com/JCesar206" target="_blank" class="hover:underline">GitHub</a>
          <a href="https://www.linkedin.com/in/jcesar206" target="_blank" class="hover:underline">LinkedIn</a>
          <a href="mailto:jcesar206@hotmail.com" class="hover:underline">Hotmail</a>
          <a href="mailto:jcesary06@gtmail.com" class="hover:underline">Gmail</a>
        </div>
        <p class="text-sm">&copy; {{ new Date().getFullYear() }} Personal Finance | Juls | All right reserved.</p>
      </footer>
    </div>
  </div>
</template>