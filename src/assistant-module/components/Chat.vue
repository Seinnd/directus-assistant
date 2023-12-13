<template>
    <div id="module-assistant-chat">
        <div class="module-assistant-chat_container">
            <chat-message v-for="message in conversation" :key="message.id" :message="message" />
        </div>
        <chat-bar key="chatbar" :disabled="sendingMessage" @send-message="onSendMessage($event)" />
    </div>
</template>

<style type="css">
main > #module-assistant-chat {
    display: flex;
    flex-direction: column;
    flex: 1;
    padding: var(--content-padding) 0 0 0 !important;
    margin: var(--theme--form--column-gap) auto 0 auto !important;
    box-sizing: border-box;
    height: calc(100% - (61px + (var(--theme--form--column-gap) + var(--content-padding))));
    margin-bottom: 0!important;
}
main > #module-assistant-chat > .module-assistant-chat_container {
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    padding: calc(var(--content-padding) * 2) var(--content-padding);
    gap: var(--theme--form--column-gap);
    flex: 1;
    overflow-x: hidden;
    overflow-y: auto;
}
</style>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { useApi } from '@directus/extensions-sdk'
import { Message } from '../interfaces'
import ChatMessage from './ChatMessage.vue';
import ChatBar from './ChatBar.vue';

export default defineComponent({
    components: { ChatMessage, ChatBar },
    setup() {
        const api = useApi();
        const sendingMessage = ref(false);
        const conversation = ref<Message[]>([]);

        async function onSendMessage(message: string) {
            if (sendingMessage.value) return;
            sendingMessage.value = true;
            conversation.value.push({ content: message, created_at: new Date().toString(), id: -1, role: 'user', user: '' })
            const response = await api.post('/assistant/send', { content: message })
            sendingMessage.value = false;
            conversation.value.splice(conversation.value.length - 1, 1);
            const { ok, data } = response.data;
            if (ok) {
                conversation.value.push(data.user_message);
                for (const message of data.assistant_messages)
                    conversation.value.push(message);
            }
        }

        async function loadMessages() {
            const response = await api.get('/assistant/messages');
            const { ok, data } = response.data;
            if (ok) conversation.value = data;
        }

        loadMessages();

        return {
            sendingMessage,
            onSendMessage,
            conversation
        }
    }
})
</script>
