<script lang="tsx">
import { defineComponent, ref, onMounted, onUnmounted, onUpdated } from 'vue'

export default defineComponent({
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Chat',
  setup() {
    /** Принятые сообщения */
    const receivedMessages = ref<Array<{ text: string; senderId: string }>>([])
    const input = ref<string>('')
    const ws = ref<WebSocket | null>(null)
    /** Оправленные текущим пользователем сообщение */
    const inputMessage = ref<Array<{ text: string; senderId: string }>>([])

    // Получаем или создаем уникальный идентификатор для текущего пользователя
    let currentUser = localStorage.getItem('chatUserId')
    if (!currentUser) {
      currentUser = `user_${Math.random().toString(36).substr(2, 9)}`
      localStorage.setItem('chatUserId', currentUser)
    }

    const checkWebSocketConnection = (): boolean => {
      return !!ws.value
    }

    const addCurrentUserMessage = (mess: { text: string; senderId: string }): void => {
      inputMessage.value.push(mess)
    }

    const checkInputIsNotNull = (): boolean => {
      return input.value.trim() !== ''
    }

    const sendMessage = () => {
      if (checkWebSocketConnection() && checkInputIsNotNull()) {
        const message = { text: input.value, senderId: currentUser }
        ws.value?.send(JSON.stringify(message))
        input.value = ''
        addCurrentUserMessage(message)
        console.log('message.text: ', message.text)
      }
    }

    const handleKeyPress = (event: KeyboardEvent): void => {
      if (event.code === 'Enter') sendMessage()
    }

    onMounted(() => {
      const websocket = new WebSocket('ws://localhost:8080')

      websocket.onopen = () => {
        console.log('WebSocket Client Connected: ', websocket)
      }

      websocket.onmessage = async (event) => {
        const message = JSON.parse(await event.data.text())
        receivedMessages.value.push(message)
        console.log('Принятое сообщение: ', message)
      }

      websocket.onclose = () => {
        console.log('WebSocket Client Disconnected')
      }

      ws.value = websocket

      return () => {
        websocket.close()
      }
    })

    onUpdated(() => {
      console.log('receivedMessages: ', receivedMessages)
      console.log('inputMessage: ', inputMessage)
    })

    onUnmounted(() => {
      if (ws.value) {
        ws.value.close()
      }
    })

    return {
      messages: receivedMessages,
      input,
      sendMessage,
      handleKeyPress,
      inputMessage
    }
  },
  render() {
    return (
      <div class="Chat">
        <h1>Online Chat</h1>
        <div class="Chat-messages">
          {this.messages.map((message, index) => (
            <div key={index} class="message left">
              {message.text}
            </div>
          ))}
        </div>
        <div class="Chat-inputMessages">
          {this.inputMessage &&
            this.inputMessage.map((message, index) => (
              <div key={index} class="message right">
                {message.text}
              </div>
            ))}
        </div>
        <div class="Chat-input">
          <input
            type="text"
            value={this.input}
            onInput={(e) => (this.input = (e.target as HTMLInputElement).value)}
            onKeydown={this.handleKeyPress}
          />
          <button onClick={this.sendMessage} id="sendMessageButton">
            Send
          </button>
        </div>
      </div>
    )
  }
})
</script>

<style scoped>
.Chat {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.Chat-inputMessages {
  /* flex: 1; */
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.Chat-messages {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.message {
  margin: 5px;
  padding: 10px;
  border-radius: 10px;
  max-width: 70%;
  word-wrap: break-word;
}

.message.left {
  align-self: flex-start;
  background-color: #d3d3d3;
}

.message.right {
  align-self: end;
  background-color: #add8e6;
}

.Chat-input {
  display: flex;
  align-items: center;
  padding: 10px;
}

.Chat-input input {
  flex: 1;
  margin-right: 10px;
}

.Chat-input button {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: #fff;
  cursor: pointer;
  border-radius: 5px;
}

.Chat-input button:hover {
  background-color: #0056b3;
}
</style>
