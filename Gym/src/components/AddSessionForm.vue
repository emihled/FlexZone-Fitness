<template>
  <form @submit.prevent="handleSubmit" class="form">
    <h2>Add New Class</h2>

    <div class="input-group">
      <label>Class Name</label>
      <input v-model="name" type="text" placeholder="Enter class name"/>
    </div>

    <div class="input-group">
      <label>Coach</label>
      <input v-model="coach" type="text" placeholder="Enter coach name"/>
    </div>

    <div class="input-group">
      <label>Date</label>
      <input v-model="date" type="date"/>
    </div>

    <div class="input-group">
      <label>Time</label>
      <input v-model="time" type="time"/>
    </div>

    <div class="input-group">
      <label>Capacity</label>
      <input v-model.number="capacity" type="number" min="1" placeholder="Enter capacity"/>
    </div>

    <p v-if="error" class="error">{{ error }}</p>
    <button type="submit">Add Class</button>
  </form>
</template>

<script setup>
import { ref } from "vue"
const emit = defineEmits(["add-session"])

const name = ref(""), coach = ref(""), date = ref(""), time = ref(""), capacity = ref(null)
const error = ref("")

const handleSubmit = () => {
  if (!name.value || !coach.value || !date.value || !time.value || !capacity.value) {
    error.value = "All fields are required."
    return
  }
  emit("add-session", { id: Date.now(), name: name.value, coach: coach.value, date: date.value, time: time.value, capacity: capacity.value })
  name.value = ""; coach.value = ""; date.value = ""; time.value = ""; capacity.value = null; error.value = ""
}
</script>

<style>
.form {
  width: 100%;
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(15px);
  padding: 25px;
  margin-bottom: 30px;
  border-radius: 25px;
  border: 1px solid rgba(255,255,255,0.2);
  box-shadow: 0 8px 32px rgba(0,0,0,0.25);
  transition: transform 0.2s;
}
.form:hover { transform: translateY(-3px); }

.input-group input {
  width: 100%;
  padding: 8px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255,255,255,0.3);
  background: rgba(255,255,255,0.05);
  color: inherit;
  backdrop-filter: blur(5px);
  transition: border 0.2s, background 0.2s;
}
.input-group input:focus {
  border: 1px solid #3498db;
  background: rgba(255,255,255,0.15);
  outline: none;
}

button {
  padding: 10px 20px;
  border-radius: 20px;
  border: none;
  background: rgba(255, 255, 255, 0.2);
  color: inherit;
  font-weight: bold;
  cursor: pointer;
  backdrop-filter: blur(5px);
  transition: background 0.3s, transform 0.2s;
}
button:hover {
  background: rgba(255,255,255,0.3);
  transform: translateY(-2px);
}

.error { color: #e74c3c; margin-bottom: 10px; }
</style>
