<template>
  <v-card text="Chat with the Assistant" color="black"></v-card>
  <v-card id="chat" ref="chatbox">
    <div v-for="(message, index) in messages" :key="index">
      <div v-if="message.role === 'user'">
        <div class="d-flex flex-row mb-6">
          <v-col cols="10">
            <v-card elevation="16" color="light-blue" prepend-icon="mdi-account" subtitle="You">
              <v-card-text>
                <div v-html="message.getContent()"></div>
              </v-card-text>
            </v-card>
          </v-col>
        </div>
      </div>
      <div v-else-if="message.role === 'assistant'">
        <div class="d-flex flex-row-reverse mb-6">
          <v-col cols="10">
            <v-card elevation="16" color="white" prepend-icon="mdi-robot" subtitle="Assistant">
              <v-card-text id="assistant-content">
                <div v-html="message.getContent()"></div>
              </v-card-text>
            </v-card>
          </v-col>
        </div>
      </div>
    </div>
    <div class="d-flex flex-row-reverse mb-6">
      <v-col cols="8" v-if="isLoading">
        <v-progress-circular color="primary" indeterminate></v-progress-circular>
      </v-col>
    </div>
    <v-footer absolute id="footer" color="#f4f5ce" border>
      <v-col class="text-center mt-4" cols="10">
        <v-textarea ref="promptField" label="Prompt" @keyup.enter="onSubmit"></v-textarea>
      </v-col>
      <v-col class="text-center">
        <v-col cols="auto">
          <v-btn icon="mdi-send" size="x-large" color="blue" @click="onSubmit"></v-btn>
        </v-col>
      </v-col>
    </v-footer>
  </v-card>
</template>

<script>
import axios from 'axios';

class PromptMessage
{
  constructor(content, role)
  {
    this.content = content;
    this.role = role;
  }

  setContent(content)
  {
    this.content = content;
  }

  setRole(role)
  {
    this.role = role;
  }

  getContent()
  {
    return this.content;
  }
}


class AgentRequest
{
  constructor(prompt, agentId, agentAlias, sessionId)
  {
    this.prompt = prompt;
    this.agentId = agentId;
    this.agentAlias = agentAlias;
    this.sessionId = sessionId;
  }

}

export default {
  data()
  {
    return {
      messages: [],
      isLoading: false
    };
  },
  methods: {
    async onSubmit(event)
    {
      let prompt = this.$refs.promptField.value;

      this.messages.push(new PromptMessage(prompt, "user"));
      this.isLoading = true;
      this.$refs.promptField.reset();
      var chatBox = document.getElementById("chat");
      console.log(chatBox.scrollHeight);

      var scrollAdjuster = setInterval(() =>
        chatBox.scrollTo({
          top: chatBox.scrollHeight,
          left: 0,
          behavior: "smooth"
        }), 200);

      await this.fetchStreamedResponse(prompt);

      clearInterval(scrollAdjuster);

      setTimeout(() =>
        chatBox.scrollTo({
          top: chatBox.scrollHeight,
          left: 0,
          behavior: "smooth"
        }), 200);

      this.isLoading = false;
    },
    async fetchStreamedResponse(prompt)
    {
      var request = new AgentRequest(prompt, "HICRJN5VKM", "JBHUS8IQMD", "12321321asdad");

      const response = await fetch('/api/erm-ai/rest/v1/api/chatbot/agent/prompt', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(request)
      });
      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let chunk;
      let assistantMessage = new PromptMessage("", "assistant");
      this.messages.push(assistantMessage);

      while (!(chunk = await reader.read()).done)
      {
        const text = decoder.decode(chunk.value, { stream: true });
        assistantMessage.setContent(assistantMessage.getContent() + text);
        // Ensure Vue reactivity updates the UI
        this.messages = [...this.messages];
      }
    },
    copyToClipboard(text)
    {
      console.log("Copying to clipboard: " + text);
      navigator.clipboard.writeText(text);
    }
  },
  mounted()
  {
    this.messages = [];
    this.isLoading = false;
  }
};
</script>


<style>
#footer {
  position: fixed;
  bottom: 0;
  width: 100%;
}

#chat {
  width: 100vw;
  height: 80vh;
  overflow-y: auto;
}

#assistant-content {
  margin: 10px;
}
</style>
