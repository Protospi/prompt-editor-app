<template>
  <q-page class="copilot-page">
    <div class="header-section">
      <div class="text-h3 q-mb-md text-center" style="color: #6467F2;">Live Copilot</div>
      <div class="text-subtitle1 q-mb-lg text-center text-grey-7">
        AI assistant to help you analyze your data and provide intelligent insights
      </div>
    </div>
    <div class="copilot-container">
      <copilot-component />
    </div>
  </q-page>
</template>

<script setup lang="ts">
import CopilotComponent from 'components/CopilotComponent.vue';
</script>

<style scoped>
.copilot-page {
  padding: 1.5rem;
  min-height: 100vh;
  background-color: white;
  width: 100%;
}

.header-section {
  margin-bottom: 1rem;
}

.copilot-container {
  margin: 0 auto;
  width: 95%;
  max-width: 800px;
  height: 80vh;
}
</style> 