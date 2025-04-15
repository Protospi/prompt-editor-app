<template>
  <div class="copilot-component">
    <q-card class="shadow-0 g-pa-24 g-border-radius-10 g-card-medium full-height">
      <!-- Main content card -->
      <q-card class="bg-white q-pa-md full-height" style="display: flex; flex-direction: column;">
        <!-- Header -->
        <div class="row items-center justify-between q-mb-md">
          <div class="text-h6">Live Copilot</div>
          <div class="row justify-end">
            <q-btn flat round color="secondary" @click="startNewConversation">
              <q-icon name="add_comment" size="24px"></q-icon>
            </q-btn>
          </div>
        </div>

        <q-separator class="q-mb-md" />

        <!-- Chat area -->
        <div class="chat-area q-mb-md" ref="messagesContainer" style="flex-grow: 1; overflow-y: auto;">
          <!-- Welcome message in a bubble - always visible -->
          <div class="message-container welcome-message">
            <div class="avatar-circle assistant-avatar">
              <div class="assistant-avatar-inner">ST</div>
            </div>
            <div class="message-bubble assistant-bubble">
              <div class="welcome-text">
                How can I help you about this conversation? You can send me a message or pick any of the suggestions below
              </div>
            </div>
          </div>
          
          <!-- Display chat messages -->
          <div v-for="(message, index) in messages" :key="index" class="message-container" :class="message.sender === 'user' ? 'user-message-container' : ''">
            <!-- Avatar circle with initials or icon -->
            <div class="avatar-circle" :class="message.sender === 'user' ? 'user-avatar' : 'assistant-avatar'">
              <div v-if="message.sender === 'user'" class="user-initials">PL</div>
              <div v-else class="assistant-avatar-inner">ST</div>
            </div>
            
            <!-- Message bubble -->
            <div class="message-bubble" :class="message.sender === 'user' ? 'user-bubble' : 'assistant-bubble'">
              {{ message.text }}
            </div>
          </div>

          <!-- Thinking indicator -->
          <div v-if="thinking" class="message-container">
            <div class="avatar-circle assistant-avatar">
              <div class="assistant-avatar-inner">ST</div>
            </div>
            <div class="message-bubble assistant-bubble">
              <span class="thinking-indicator">
                <span class="dot"></span>
                <span class="dot"></span>
                <span class="dot"></span>
              </span>
            </div>
          </div>

          <!-- Suggested message buttons -->
          <div v-if="messages.length === 0" class="column q-gutter-y-md q-mt-md" style="width: 70%; margin-left: 60px;">
            <q-btn 
              v-for="(suggestion, index) in suggestedMessages" 
              :key="index"
              class="full-width text-left suggestion-btn" 
              @click="sendSuggestedMessage(suggestion)"
              style="height: 36px; padding: 0 12px; border-radius: 10px;"
            >
              {{ suggestion }}
            </q-btn>
          </div>

          <!-- Display follow-up questions if messages is not empty -->
          <div v-if="messages.length > 0" class="message-container">
            <div class="avatar-circle assistant-avatar">
              <div class="assistant-avatar-inner">ST</div>
            </div>
            <div class="follow-up-container">
              <div class="follow-up-label">Follow-up Questions:</div>
              <div class="column q-gutter-y-md q-mt-sm">
                <q-btn 
                  v-for="(question, index) in followUpQuestions" 
                  :key="index"
                  class="full-width text-left suggestion-btn" 
                  @click="sendSuggestedMessage(question)"
                  style="height: 36px; padding: 0 12px; border-radius: 10px;"
                >
                  {{ question }}
                </q-btn>
              </div>
            </div>
          </div>
        </div>

        <q-separator class="q-mb-md" />

        <!-- Input area and footer text -->
        <div class="input-footer-area">
          <!-- Input area -->
          <div class="input-area q-mb-sm">
            <q-input
              v-model="userInput"
              outlined
              dense
              placeholder="Ask me anything about the data"
              @keyup.enter="sendMessage"
              rounded
            >
              <template v-slot:after>
                <q-btn 
                  round 
                  dense 
                  flat 
                  color="secondary" 
                  @click="sendMessage"
                >
                  <template v-if="thinking">
                    <q-spinner color="secondary" size="1em" :thickness="8" />
                  </template>
                  <template v-else>
                    <q-icon name="send" />
                  </template>
                </q-btn>
              </template>
            </q-input>
          </div>

          <!-- Footer text -->
          <div class="text-caption text-grey text-center">
            The agent is powered by AI, so please verify for errors and don't share sensitive information.
            Your data will be used according to SmartTalks.ai Privacy Policy.
          </div>
        </div>
      </q-card>
    </q-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, nextTick } from 'vue';

// Types
interface Message {
  text: string;
  sender: 'user' | 'copilot';
  timestamp: Date;
}

// Refs
const userInput = ref('');
const messages = ref<Message[]>([]);
const thinking = ref(false);
const messagesContainer = ref<HTMLElement | null>(null);

// Hardcoded suggested messages
const suggestedMessages = ref([
  'What are the general sentiments?',
  'How are user interactions?',
  'What are the most common keywords?'
]);

// Hardcoded follow-up questions
const followUpQuestions = ref([
  'Can you provide more details?',
  'How does this compare to last month?',
  'What actions should I take?'
]);

// Initial setup
onMounted(() => {
  // We'll leave this empty for now as the welcome message is always shown
});

// Methods
function addMessage(text: string, sender: 'user' | 'copilot') {
  messages.value.push({
    text,
    sender,
    timestamp: new Date()
  });
  
  // Scroll to bottom on next tick
  void nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
  });
}

function sendMessage() {
  const text = userInput.value.trim();
  if (!text) return;
  
  // Add user message
  addMessage(text, 'user');
  userInput.value = '';
  
  // Simulate thinking
  thinking.value = true;
  
  // Simulate AI response (would be replaced with actual API call)
  setTimeout(() => {
    thinking.value = false;
    
    // Mock response based on input
    let response = '';
    if (text.toLowerCase().includes('hello') || text.toLowerCase().includes('hi')) {
      response = 'Hello Pedro! How can I help you today?';
    } else if (text.toLowerCase().includes('help')) {
      response = 'I can help with analyzing data, answering questions, or providing insights. What specifically do you need assistance with?';
    } else {
      response = `I understand you're asking about "${text}". I'm analyzing the data now. In a full implementation, I would provide detailed analytics and insights based on your query.`;
    }
    
    addMessage(response, 'copilot');
  }, 1500);
}

function sendSuggestedMessage(text: string) {
  userInput.value = text;
  sendMessage();
}

function startNewConversation() {
  messages.value = [];
  userInput.value = '';
  
  // Scroll back to top to see the welcome message
  void nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = 0;
    }
  });
}

// Watch for new messages and scroll to bottom
watch(messages, () => {
  void nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
  });
}, { deep: true });
</script>

<style scoped>
.copilot-component {
  height: 70vh;
  width: 100%;
}

.full-height {
  height: 100%;
}

.message-container {
  display: flex;
  margin-bottom: 16px;
  align-items: flex-start;
}

.welcome-message {
  padding-top: 8px;
  margin-bottom: 24px;
}

.welcome-text {
  font-size: 14px;
  line-height: 1.4;
}

.user-message-container {
  flex-direction: row-reverse;
}

.avatar-circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-shrink: 0;
  margin: 0 12px;
}

.assistant-avatar {
  background-color: #6467F2;
  color: white;
}

.assistant-avatar-inner {
  font-size: 14px;
  font-weight: bold;
}

.user-avatar {
  background-color: #eaeafa;
  color: #6467F2;
}

.user-initials {
  font-size: 14px;
  font-weight: bold;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 18px;
  max-width: 70%;
}

.user-bubble {
  background-color: #f5f5f5;
  border-top-right-radius: 4px;
  color: black;
}

.assistant-bubble {
  background-color: #eaeafa;
  border-top-left-radius: 4px;
  color: black;
}

.follow-up-container {
  background-color: white;
  border-radius: 12px;
  padding: 12px;
  max-width: 70%;
}

.follow-up-label {
  font-weight: bold;
  margin-bottom: 8px;
}

/* Thinking animation */
.thinking-indicator {
  display: inline-flex;
  gap: 4px;
  align-items: center;
  background-color: #dfdff9;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #6467F2;
  display: inline-block;
  animation: pulse 1.5s infinite;
}

.dot:nth-child(2) {
  animation-delay: 0.2s;
}

.dot:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(0.8);
    opacity: 0.5;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .copilot-component {
    height: 80vh;
  }
  
  .message-bubble {
    max-width: 80%;
  }
}

.suggestion-btn {
  background-color: #eaeafa;
  color: #6467F2;
  border: none;
  font-weight: 500;
}
</style> 