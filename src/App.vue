<template>
  <main>
    <div class="chat-container">
      <h1>Chat</h1>
      <div class="messages" ref="messagesScroll">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="message"
          :class="msg.sender"
        >
          {{ msg.text }}
        </div>
      </div>
      <div class="input">
        <input
          type="text"
          placeholder="Type a message..."
          v-model="newMessage"
          :readonly="loading"
          @keyup.enter="callAgent"
        />
        <button v-if="!loading" @click="callAgent">Enviar</button>
        <button v-else>
          <span class="spinner"></span>
        </button>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      messages: [],
      newMessage: "",
    };
  },
  methods: {
    addMessage({ msg, sender = "user" }) {
      if (msg.trim() == "") return;

      this.messages.push({ text: msg, sender });
      this.$nextTick(() => {
        this.scrollToBottom();
      });
    },
    callAgent() {
      if (this.newMessage.trim() === "") return;
      const msg = this.newMessage;
      this.newMessage = "";
      this.addMessage({ msg, sender: "user" });

      this.loading = true;

      fetch("https://cloudwalk-agent-swarm-lnrdrodri.onrender.com/api/agent", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          message: msg,
          user_id: "1",
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          const msg = data?.reply || "Não foi possivel obter a resposta.";
          this.addMessage({
            msg,
            sender: "agent",
          });
        })
        .catch((error) => {
          console.error("Error:", error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    scrollToBottom() {
      const messages = this.$refs.messagesScroll;
      if (messages) {
        messages.scrollTop = messages.scrollHeight;
      }
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

main {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background-color: #0e0e0e;
  color: #fff;
  height: 100vh;
  display: grid;
  place-items: center;
}

.chat-container {
  display: flex;
  flex-direction: column;
  width: 60vw;
  height: 80vh;
  margin: 0 auto;
  border: 1px solid #333;
  border-radius: 8px;
}

.chat-container > h1 {
  padding: 8px 16px;
  border-bottom: 1px solid #333;
}

.chat-container .input {
  height: 10vh;
  display: flex;
}

.chat-container .input > input {
  flex: 1;
  padding: 0 16px;
  border: none;
  outline: none;
  font-size: 16px;
  background: #1e1e1e;
  color: #fff;
}

.chat-container .input > button {
  width: 100px;
  border: none;
  background: #333;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
}

.messages {
  flex: 1;
  padding: 16px;
  overflow-y: scroll;
  display: flex;
  flex-direction: column;
}

.messages::-webkit-scrollbar {
  width: 8px;
  background: transparent;
}

.messages::-webkit-scrollbar-thumb {
  background: #333;
  border-radius: 8px;
}

.message {
  padding: 8px 16px;
  background: #1e1e1e;
  border-radius: 8px;
  border: 1px solid #333;
  margin-bottom: 8px;
  max-width: 70%;
  word-wrap: break-word;
  align-self: start;
}

.message.agent {
  background: black;
  align-self: end;
}

.spinner {
  display: inline-block;
  animation: spin 1s linear infinite;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: #fff;
  width: 16px;
  height: 16px;
  line-height: 0;
  vertical-align: middle;
  cursor: wait;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@media (max-width: 1023px) {
  .chat-container {
    width: 100vw;
    height: 100vh;
    border: none;
  }

  .message {
    max-width: 80%;
  }
}
</style>
