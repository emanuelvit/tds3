<script setup>
import { ref, computed, watch, nextTick, onMounted } from "vue";

const STORAGE_MESSAGES = "chat_messages";
const STORAGE_USERNAME = "chat_username";

function loadFromStorage(key, fallback) {
  try {
    const raw = localStorage.getItem(key);
    return raw ? JSON.parse(raw) : fallback;
  } catch (e) {
    return fallback;
  }
}

function saveToStorage(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (e) {
    // localStorage indisponível ou cheio: ignora
  }
}

const username = ref(loadFromStorage(STORAGE_USERNAME, ""));
const newMessage = ref("");
const messages = ref(loadFromStorage(STORAGE_MESSAGES, []));
const messagesBox = ref(null);
let nextId = messages.value.reduce((max, m) => Math.max(max, m.id), 0) + 1;

const canSend = computed(
  () => username.value.trim() !== "" && newMessage.value.trim() !== ""
);

watch(messages, (value) => saveToStorage(STORAGE_MESSAGES, value), { deep: true });
watch(username, (value) => saveToStorage(STORAGE_USERNAME, value));

function scrollToBottom() {
  if (messagesBox.value) {
    messagesBox.value.scrollTop = messagesBox.value.scrollHeight;
  }
}

onMounted(scrollToBottom);

async function sendMessage() {
  if (!canSend.value) return;
  messages.value.push({
    id: nextId++,
    author: username.value.trim(),
    text: newMessage.value.trim(),
  });
  newMessage.value = "";
  await nextTick();
  scrollToBottom();
}
</script>

<template>
  <div class="chat">
    <header>
      <label for="username">Nome de usuário</label>
      <input id="username" v-model="username" placeholder="Digite seu nome..." />
    </header>

    <div class="messages" ref="messagesBox">
      <p v-if="messages.length === 0" class="empty">Nenhuma mensagem ainda.</p>
      <div v-for="msg in messages" :key="msg.id" class="message">
        <div class="author">{{ msg.author }}</div>
        <div class="text">{{ msg.text }}</div>
      </div>
    </div>

    <form @submit.prevent="sendMessage">
      <input v-model="newMessage" placeholder="Digite uma mensagem..." />
      <button type="submit" :disabled="!canSend">Enviar</button>
    </form>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  background: #f0f2f5;
  margin: 0;
  display: flex;
  justify-content: center;
  padding: 24px 12px;
}
</style>

<style scoped>
.chat {
  width: 100%;
  max-width: 520px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  height: 80vh;
  overflow: hidden;
}

header {
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
}

header label {
  display: block;
  font-size: 13px;
  color: #555;
  margin-bottom: 6px;
}

input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 15px;
}

input:focus {
  outline: 2px solid #4f46e5;
  border-color: transparent;
}

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.empty {
  color: #9ca3af;
  text-align: center;
  margin-top: 20px;
}

.message {
  background: #eef2ff;
  border-radius: 10px;
  padding: 8px 12px;
  max-width: 85%;
  align-self: flex-start;
  word-wrap: break-word;
}

.message .author {
  font-weight: 600;
  font-size: 13px;
  color: #4f46e5;
}

.message .text {
  margin-top: 2px;
  white-space: pre-wrap;
}

form {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  border-top: 1px solid #e5e7eb;
}

button {
  padding: 10px 18px;
  border: none;
  border-radius: 8px;
  background: #4f46e5;
  color: #fff;
  font-size: 15px;
  cursor: pointer;
}

button:disabled {
  background: #a5b4fc;
  cursor: not-allowed;
}

.text{
    color: black;
}
</style>
