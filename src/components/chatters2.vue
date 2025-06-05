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
  </v-card>

  <v-footer id="footer" class="d-flex flex-column" color="#f4f5ce">
    <div class="px-4 py-2 w-100 rounded-lg">
      <v-file-input v-model="files" label="Upload a file" chips multiple density="compact"></v-file-input>
    </div>
    <div class="d-flex w-100 align-center px-4 py-1">
      <v-textarea ref="promptField" label="Prompt" @keyup.enter="onSubmit"></v-textarea>
      <v-btn icon="mdi-send" size="x-large" color="blue" @click="onSubmit"></v-btn>
    </div>
    <v-switch
      v-model="serviceSwitch"
      :label="`Switch: ${serviceSwitch}`"
      false-value="ERM"
      true-value="AI Service"
    ></v-switch>

  </v-footer>

  <!-- <v-footer absolute id="footer" color="#f4f5ce" border>
      <v-row>
        <v-col cols="10">
          <v-textarea ref="promptField" label="Prompt" @keyup.enter="onSubmit"></v-textarea>
        </v-col>
        <v-col class="text-center">
          <v-col cols="auto">
            <v-btn icon="mdi-send" size="x-large" color="blue" @click="onSubmit"></v-btn>
          </v-col>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="10">
          <v-file-input label="Uplaod a file" chips multiple></v-file-input>
        </v-col>
      </v-row>

    </v-footer> -->
</template>
<script>
import { ref } from 'vue'




class PromptMessage {
  constructor(content, role) {
    this.content = content;
    this.role = role;
  }

  setContent(content) {
    this.content = content;
  }

  setRole(role) {
    this.role = role;
  }

  getContent() {
    return this.content;
  }
}


class AgentRequest {
  constructor(prompt, agentId, agentAlias, sessionId) {
    this.prompt = prompt;
    this.agentId = agentId;
    this.agentAlias = agentAlias;
    this.sessionId = sessionId;
  }

}

export default {
  data() {
    return {
      messages: [],
      isLoading: false,
      files: ref([]),
      serviceSwitch: ref("ERM")
    };
  },
  methods: {
    async onSubmit(event) {
      let prompt = this.$refs.promptField.value;

      this.messages.push(new PromptMessage(prompt, "user"));
      this.isLoading = true;
      this.$refs.promptField.reset();
      var chatBox = document.getElementById("chat");

      console.log("files: ", this.files);


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

      this.isLoading = false; y
    },
    async fetchStreamedResponse(prompt) {

    var request = new AgentRequest(prompt, "HICRJN5VKM", "8WEVCCHTM1", "12313asdsada");

      const formData = new FormData();
      // formData.append('file', fileInput.files[0]);
      formData.append('request', new Blob([JSON.stringify(request)], {
        type: 'application/json'
      }));


      if (this.files !== undefined && this.files.length > 0) {
        for (let i = 0; i < this.files.length; i++) 
        {
          formData.append('files', this.files[i]);
        }

      }

      const myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");
      myHeaders.append("Authorization",'Basic cHJvdGVjaHQuc3VwcG9ydDppTUFpSkx2YU1FamlxckNUTkhTR1hPVDZuK2xaVno2VDJ2VzdBMGhsTmtpRmVyM2p0dE1SdVh2SGV6TzlJaDhq');

      console.log("Service switch: " + this.serviceSwitch);

      const requestBody  =  this.serviceSwitch === "ERM"
      ? JSON.stringify({ "prompt": prompt})
        : formData;


      const requestUri = this.serviceSwitch === "ERM" 
      ? "/api/camilla/rest/api/copilot/chat/stream" 
      : "/api/erm-ai/rest/v1/api/chatbot/agent/stream";

      const response = await fetch(requestUri, {
        method: 'POST',
        headers: myHeaders,
        body: requestBody
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let chunk;
      let assistantMessage = new PromptMessage("", "assistant");
      this.messages.push(assistantMessage);

      while (!(chunk = await reader.read()).done) {
        const text = decoder.decode(chunk.value, { stream: true });
        assistantMessage.setContent(assistantMessage.getContent() + text);
        console.log("Assistant message: " + assistantMessage.getContent());
        // Ensure Vue reactivity updates the UI
        this.messages = [...this.messages];
      }
    },
    copyToClipboard(text) {
      console.log("Copying to clipboard: " + text);
      navigator.clipboard.writeText(text);
    }
  },
  mounted() {
    this.messages = [];
    this.isLoading = false;
  }
};
</script>




<style>
#footer {
  /* position: fixed;
  bottom: 0; */
}

#chat {
  width: 100vw;
  height: 80vh;
  height: 72vh;
  overflow-y: auto;
}

#assistant-content {
  margin: 10px;
}
</style>
