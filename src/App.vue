<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";
const socket = io("http://localhost:3001");
const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });
  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});
const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};
const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, (response) => {
    messageText.value = "";
  });
};
let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });

  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>What's your name ?</label>
        <input v-model="name" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="message" v-for="message in messages">
        <div class="sender-info" >
          <span class="sender-name">{{ message.name }} </span>
          <span class="message-timestamp">{{ message.hours }} :{{message.minutes}}</span>
        </div>
        <div class="message-content">
          <p>{{ message.text }}</p>
        </div>
      </div>

      <div v-if="typingDisplay" class="logo is-animetion">{{ typingDisplay }}</div>
      <hr />
      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input style="margin: 0 24px" type="text" v-model="messageText" @input="emitTyping"  placeholder="Type your message..."/>

          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
@import "./assets/base.css";
.chat {
  padding: 20px;
  height: 87vh;
}
.chat-container {
  max-width: 600px;
  margin: 20px auto;
  padding: 10px;
  border: 1px solid #ccc;
}

/* Style for each individual message */
.message {
  margin-bottom: 20px;
  padding: 10px;
  border-radius: 8px;
  background-color: #f0f0f0;
}

/* Style for sender information */
.sender-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
  font-weight: bold;
}

.sender-name {
  color: #333;
}

.message-timestamp {
  color: #555;
  font-size: 0.9em;
}

/* Style for message content */
.message-content {
  color: #333;
}

.message-content p {
  margin: 0;
}

.messages-container {
  flex: 1;
}

.message-input {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

/* Style for input field */
.message-input input[type="text"] {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  outline: none; /* Remove default focus outline */
}

/* Style for send button */
.message-input button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.message-input button:hover {
  background-color: #0056b3;
}
.logo.is-animetion {
  margin-top: 0.6em;
}

.logo.is-animetion{
  display: inline-block;
  animation: wave-text 1s ease-in-out infinite;
}


@keyframes wave-text{
  00%{
    transform: translateY(0em);
  }
  60%{
    transform: translateY(-0.6em);
  }
  100%{
    transform: translateY(0em);
  }
}
</style>
