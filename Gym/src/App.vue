<template>
  <div :class="['app-wrapper', darkMode ? 'dark' : 'light']">
    <header>
      <button @click="darkMode = !darkMode" class="mode-toggle">
        {{ darkMode ? 'Light Mode' : 'Dark Mode' }}
      </button>
      <h1>FlexZone Fitness Class Schedule</h1>
    </header>

    <main>
      <AddSessionForm @add-session="addSession" />

      <!-- SEARCH & FILTER -->
      <div class="search-bar">
        <input
          v-model="searchCoach"
          type="text"
          placeholder="Search by coach…"
        />
        <select v-model="selectedCoach">
          <option value="">Filter by coach</option>
          <option v-for="coach in coachList" :key="coach" :value="coach">
            {{ coach }}
          </option>
        </select>
        <button @click="applySearch">Search</button>
        <button v-if="searchCoach || selectedCoach" @click="clearSearch" class="clear-btn">
          Clear
        </button>
      </div>

      <!-- SORT -->
      <div class="sort-bar">
        <label>Sort by:</label>
        <select v-model="sortField">
          <option value="">None</option>
          <option value="name">Name</option>
          <option value="date">Date</option>
          <option value="capacity">Capacity</option>
        </select>
        <button @click="sortAsc = !sortAsc">
          {{ sortAsc ? 'Asc' : 'Desc' }}
        </button>
      </div>

      <h2>Total Sessions: {{ filteredSessions.length }}</h2>

      <SessionList
        :sessions="filteredSessions"
        @delete-session="deleteSession"
      />
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from "vue"
import AddSessionForm from "./components/AddSessionForm.vue"
import SessionList from "./components/SessionList.vue"

const sessions = ref([])

// SEARCH / FILTER
const searchCoach = ref("")
const selectedCoach = ref("")
const activeFilter = ref("")
window.activeFilter = activeFilter

// SORT
const sortField = ref("")
const sortAsc = ref(true)

// DARK MODE
const darkMode = ref(false)

// ADD / DELETE
const addSession = (session) => { sessions.value.push(session) }
const deleteSession = (id) => { sessions.value = sessions.value.filter(s => s.id !== id) }

// COACH LIST
const coachList = computed(() => [...new Set(sessions.value.map(s => s.coach))])

// FILTER + SORT
const filteredSessions = computed(() => {
  const textFilter = activeFilter.value.toLowerCase()
  const dropFilter = selectedCoach.value.toLowerCase()

  let filtered = sessions.value.filter(s => {
    const coach = s.coach.toLowerCase()
    const matchText = textFilter ? coach.includes(textFilter) : true
    const matchDropdown = dropFilter ? coach === dropFilter : true
    return matchText && matchDropdown
  })

  if (sortField.value) {
    filtered.sort((a, b) => {
      let valA = a[sortField.value]
      let valB = b[sortField.value]
      if (sortField.value === "date") {
        valA = new Date(valA)
        valB = new Date(valB)
      }
      if (valA < valB) return sortAsc.value ? -1 : 1
      if (valA > valB) return sortAsc.value ? 1 : -1
      return 0
    })
  }
  return filtered
})

// SEARCH FUNCTIONS
const applySearch = () => { activeFilter.value = searchCoach.value }
const clearSearch = () => { searchCoach.value = ""; selectedCoach.value = ""; activeFilter.value = "" }

// DEBOUNCED SEARCH
let debounceTimer = null
watch(searchCoach, (newVal) => {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => { activeFilter.value = newVal }, 300)
})

// LOCALSTORAGE
onMounted(() => {
  const saved = localStorage.getItem("sessions")
  if (saved) sessions.value = JSON.parse(saved)
})
watch(sessions, (newVal) => localStorage.setItem("sessions", JSON.stringify(newVal)), { deep: true })
</script>

<style>
/* Shiny gradient background for full-screen effect */
.app-wrapper {
  min-height: 100vh;
  padding: 20px;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: linear-gradient(135deg, rgba(255,255,255,0.05), rgba(0,0,0,0.05));
  position: relative;
  overflow: hidden;
}
.app-wrapper::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(120deg, #ff9a9e, #fad0c4, #a18cd1, #fbc2eb);
  background-size: 400% 400%;
  animation: gradientShift 20s ease infinite;
  opacity: 0.15;
  z-index: 0;
  border-radius: 50%;
}
@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* Dark / Light text */
.app-wrapper.light { color: #222; }
.app-wrapper.dark { color: #f5f5f5; }

/* Place content above background */
header, main { position: relative; z-index: 1; width: 100%; max-width: 900px; }

/* Header glass panel */
header {
  padding: 15px 25px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(18px);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.25);
  border: 1px solid rgba(255, 255, 255, 0.2);
}
header h1 { margin-top: 10px; font-size: 2rem; color: inherit; }

/* Mode toggle button */
.mode-toggle {
  padding: 8px 16px;
  border: none;
  border-radius: 12px;
  background: rgba(255,255,255,0.2);
  color: inherit;
  font-weight: bold;
  cursor: pointer;
  backdrop-filter: blur(5px);
  transition: all 0.3s;
}
.mode-toggle:hover { transform: translateY(-2px); background: rgba(255,255,255,0.3); }

/* Search / Filter / Sort Bars */
.search-bar, .sort-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 20px;
  padding: 12px 20px;
  border-radius: 25px;
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255,255,255,0.15);
  box-shadow: 0 6px 20px rgba(0,0,0,0.15);
}
.search-bar input, .search-bar select, .sort-bar select {
  padding: 8px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255,255,255,0.3);
  background: rgba(255,255,255,0.05);
  color: inherit;
  backdrop-filter: blur(5px);
  transition: all 0.2s;
}
.search-bar input:focus, .search-bar select:focus, .sort-bar select:focus {
  border: 1px solid #3498db;
  background: rgba(255,255,255,0.15);
  outline: none;
}
.clear-btn { background: rgba(255,255,255,0.2); color: inherit; border: none; padding: 6px 12px; border-radius: 12px; cursor: pointer; }
.clear-btn:hover { background: rgba(255,255,255,0.35); }

/* Total sessions */
h2 { margin-bottom: 15px; }

/* Responsive */
@media (max-width: 600px) {
  .search-bar, .sort-bar { flex-direction: column; align-items: stretch; }
}
</style>
