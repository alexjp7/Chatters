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
    <v-switch v-model="serviceSwitch" :label="`Switch: ${serviceSwitch}`" false-value="ERM"
      true-value="AI Service"></v-switch>

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
  constructor(userPrompt, agentId, agentAlias, sessionId) {
    this.prompt = prompt;
    this.agentId = agentId;
    this.agentAlias = agentAlias;
    this.sessionId = sessionId;
  }

}

class ChatRequest { 
  constructor(userPrompt, systemPrompt) {
    this.userPrompt = userPrompt;
    this.systemPrompt = systemPrompt;
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

      // var request = new AgentRequest(prompt +". Please format your response in basic HTML and css.", "HICRJN5VKM", "8WEVCCHTM1", "12313asdsada");
      var request = new ChatRequest(prompt, "");

      const formData = new FormData();
      // formData.append('file', fileInput.files[0]);
      formData.append('request', new Blob([JSON.stringify(request)], {
        type: 'application/json'
      }));



      if (this.files !== undefined && this.files.length > 0) {
        for (let i = 0; i < this.files.length; i++) {
          formData.append('files', this.files[i]);
        }
      }

      const myHeaders = new Headers();

      if (this.serviceSwitch === "ERM") {
        myHeaders.append("Accept", "application/json");
        myHeaders.append("Content-Type", "application/json");
        myHeaders.append("Authorization", 'Basic cHJvdGVjaHQuc3VwcG9ydDppTUFpSkx2YU1FamlxckNUTkhTR1hPVDZuK2xaVno2VDJ2VzdBMGhsTmtpRmVyM2p0dE1SdVh2SGV6TzlJaDhq');

      } else {
      //  myHeaders.append("Content-Type", "application/json");
      }

      console.log("Service switch: " + this.serviceSwitch);
      console.log("headers: ", myHeaders);

      const requestBody = this.serviceSwitch === "ERM"
        ? JSON.stringify({ "prompt": request.prompt })
        : formData;
        
        // JSON.stringify({
        //   "prompts": [
        //     {
        //       "content": "We have identified a risk which is as follows <risk> Outdated Legacy System <\/risk> <risk_description> Hardware failure etc. <\/risk_description> Our organization has the following controls in place to mitigate these risks. Each control record is in the following format: [ID | Control Name | Control Description] <control> [1000002|Backup Server Tested Quarterly by IT] [1000020|DIE training|training for all employees for diversity, inclusion and equality.] [[1000041 | Monthly maintenance ], [1000057 | Temperature Control], [1000061 | Installation of software on operational systems], [1000063 | Creates and Maintains Records of System Storage Activities\\u2014Records of system storage activities are created and maintained completely and accurately in a timely manner.], [1000053 | Employee Training], [1000055 | Hazard Analysis and Critical Control Points (HACCP)], [1000065 | ISM-1334 : Interference between wireless networks], [1000081 | Segregation of duties separating the roles of processing, assessment\/approval and execution signed-off by process owner], [1000083 | Can you confirm that the organisation has an enhanced customer due diligence program and that this has been applied to any new or proposed transactions with parties, either physically present in or incorporated in, a foreign prescribed country?], [1000085 | ISM-1092 : Sending fax messages], [1000087 | SC-19 Voice Over Internet Protocol], [1000043 | Climate Policy], [1000045 | Strict Hygiene Standards], [1000047 | Proper Food Storage], [1000073 | AU-009 Protection of Audit Information], [1000075 | ISM-1299 : Personnel awareness], [1000077 | MA-2 Controlled Maintenance], [1000079 | Screening], [1000067 | ISM-1637 : Outsourced cloud services], [1000069 | A baseline configuration of information technology\/industrial control systems is created and maintained incorporating security principles (e.g. concept of least functionality)], [1000071 | Communicates Information About System Operation and Boundaries(CC2.3.8)], [1000089 | ISM-1235 : Hardening user application configurations], [1000091 | ISM-0213 : Terminating cables on patch panels], [1000093 | ISM-0687 : Approved mobile platforms], [1000049 | Regular Cleaning and Sanitation], [1000051 | Supplier Verification], [1000095 | Can you confirm that the lessor is aware of and complied with all requirements under the National Credit Code in relation to consumer leases?], [1000097 | Internal and external stakeholders are notified of incidents], [1000099 | Supplements Board Expertise], [1000101 | ISM-1000 : Perfect Forward Secrecy] <\/control> Please assess the provided controls for their relevancy to the identified risk. Consider diverse perspectives in your assessment. Always output all entries, even if they are not relevant. Your response should be in JSON format. Do not include any explanations, only provide a RFC8259 compliant JSON response following this format without deviation. Do not include markdown code blocks in your response. Remove the ```json markdown from the output. Here is the JSON Schema instance your output must adhere to: ```{ \"$schema\" : \"https:\/\/json-schema.org\/draft\/2020-12\/schema\", \"type\" : \"array\", \"items\" : { \"type\" : \"object\", \"properties\" : { \"id\" : { \"type\" : \"string\", \"description\" : \"Risk control identifier\" }, \"reason\" : { \"type\" : \"string\", \"description\" : \"Brief explanation why given risk control is relevant (or possibly not relevant)\" }, \"relevancy\" : { \"type\" : \"integer\", \"description\" : \"Relevancy of given control Use following enumerated values to quantify: Irrelevant=0, SlightlyRelevant=1, Relevant=2, HighlyRelevant=3\" } }, \"required\" : [ \"id\", \"reason\" ], \"description\" : \"A single risk control model from 'control Library' already identified by user as a relevant one to the given risk\", \"additionalProperties\" : false } }``` Your response must be strictly valid JSON with these escaping rules: - All special characters properly escaped according to RFC8259 - No unescaped control characters (0x00-0x1F) - No unescaped quotes (\") - No unescaped backslashes - No unescaped forward slashes - Unicode characters properly escaped (\\uXXXX format) - No line breaks within strings (use \\n instead) - No tabs within strings (use \\t instead). When streaming your response back - emit each control as its own stream chunk. Ths stream chunk should be a valid JSON object.",
        //       "type": "USER"
        //     }
        //   ]
        // });


      const requestUri = this.serviceSwitch === "ERM"
        ? "/api/camilla/rest/api/copilot/chat/stream"
        // : "/api/erm-ai/rest/v1/api/risk/controls/dynamic/suggest/stream/rank-linked";
        :"/api/erm-ai/rest/v1/api/chatbot/agent/stream";

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
