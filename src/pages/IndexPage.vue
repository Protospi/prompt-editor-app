<template>
  <q-page class="editor-page">
    <div class="header-section">
      <div class="text-h3 q-mb-md text-center" style="color: #6467F2;">Prompt Editor</div>
      <div class="text-subtitle1 q-mb-lg text-center text-grey-7">
        Edit your prompt in a structured way with markdown output using the AI help and save versions of your prompt in multiple languages
      </div>
    </div>
    <div class="editor-container">
      <prompt-editor />
    </div>
  </q-page>
</template>

<script setup lang="ts">
import PromptEditor from 'components/PromptEditor.vue';
</script>

<style scoped>
.editor-page {
  padding: 1.5rem;
  min-height: 100vh;
  background-color: white;
  width: 100%;
}

.header-section {
  margin-bottom: 1rem;
}

.editor-container {
  margin: 0 auto;
  width: 95%;
  max-width: 1800px;
}
</style>
