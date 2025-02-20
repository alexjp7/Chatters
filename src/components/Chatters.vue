<template>
    <v-card text="Chat with the Assistant"></v-card>
    <v-card id="chat" ref="chatbox">
        <div v-for="(message, index) in messages">
            <div v-if="message.role === 'user'">
                <div class="d-flex flex-row mb-6">
                    <v-col cols="6">
                        <v-card elevation="16" color="white" variant="outlined">
                            <v-card-title class="text-overline mb-1">
                                You
                            </v-card-title>
                            <v-card-text>
                                {{ message.getContent() }}
                            </v-card-text>
                        </v-card>
                    </v-col>
                </div>
            </div>
            <div v-else-if="message.role === 'assistant'">
                <div class="d-flex flex-row-reverse mb-6">
                    <v-col cols="6">
                        <v-card elevation="16" color="primary">
                            <v-card-title class="text-overline mb-1">
                                Assistant
                            </v-card-title>
                            <v-card-text>
                                {{ message.getContent() }}
                            </v-card-text>
                        </v-card>
                    </v-col>
                </div>
            </div>
        </div>


        <v-footer absolute id="footer">
            <v-col class="text-center" cols="10">
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
class PromptMessage {
    constructor(content, role) {
        this.content = content;
        this.role = role
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

export default {
    data: () => ({
        messages: []

    }),
    methods: {
        onSubmit(event) {
            this.messages.push(new PromptMessage(this.$refs.promptField.value, "user"));
            this.$refs.promptField.reset();
            var chatBox = document.getElementById("chat");
            console.log(chatBox.scrollHeight)
            setInterval(() =>
                chatBox.scrollTo(
                    {
                        top: chatBox.scrollHeight,
                        left: 0,
                        behavior: "smooth"
                    }
                ), 100)
        }
    },

    mounted() {
        this.messages = new Array();
        this.messages.push(new PromptMessage("Hello, what's your name?", "user"));
        this.messages.push(new PromptMessage("How are you?", "assistant"));
        this.messages.push(new PromptMessage("Hello, what's your name?", "user"));
        this.messages.push(new PromptMessage("How are you?", "assistant"));
        this.messages.push(new PromptMessage("Hello, what's your name?", "user"));
        this.messages.push(new PromptMessage("How are you?", "assistant"));
        this.messages.push(new PromptMessage("Hllo, what's your name?", "user"));
        this.messages.push(new PromptMessage("How are you?", "assistant"));

    }

}

</script>

<style>
#footer {
    position: fixed;
    bottom: 0;
    width: 100%;
    overflow: hidden;
}

#chat {
    width: 100vw;
    height: 80vh;
    overflow-y: scroll;

}
</style>
