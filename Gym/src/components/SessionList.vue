<template>
  <div>
    <h2>Scheduled Classes</h2>
    <p v-if="sessions.length === 0">No sessions found.</p>

    <transition-group name="fade" tag="ul" class="session-list">
      <li v-for="session in sessions" :key="session.id" class="session-item">
        <div>
          <strong>{{ session.name }}</strong> — <span v-html="highlight(session.coach)"></span><br>
          {{ session.date }} @ {{ session.time }}<br>
          Capacity: {{ session.capacity }}
        </div>
        <button @click="$emit('delete-session', session.id)">Delete</button>
      </li>
    </transition-group>
  </div>
</template>

<script setup>
defineProps({ sessions: Array })
const highlight = (text) => {
  const filter = window.activeFilter?.value?.toLowerCase()
  if (!filter) return text
  const regex = new RegExp(`(${filter})`, "gi")
  return text.replace(regex, "<mark>$1</mark>")
}
</script>

<style>
.session-list { list-style: none; padding: 0; }

.session-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-radius: 20px;
  margin-bottom: 12px;
  background: rgba(255,255,255,0.08);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255,255,255,0.15);
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
  transition: transform 0.2s, background 0.2s;
}
.session-item:hover {
  transform: translateY(-2px);
  background: rgba(255,255,255,0.12);
}

button {
  background: rgba(231, 76, 60, 0.3);
  color: white;
  padding: 6px 14px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  backdrop-filter: blur(5px);
  transition: background 0.3s;
}
button:hover { background: rgba(255, 25, 0, 0.5); }

mark { background: yellow; }

/* Animations */
.fade-enter-active, .fade-leave-active { transition: all 0.3s ease; }
.fade-enter-from { opacity: 0; transform: translateY(-10px); }
.fade-enter-to { opacity: 1; transform: translateY(0); }
.fade-leave-from { opacity: 1; transform: translateY(0); }
.fade-leave-to { opacity: 0; transform: translateY(10px); }
</style>
