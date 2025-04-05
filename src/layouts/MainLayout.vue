<template>
  <q-layout view="hHh lpR fFf">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
// No additional functionality needed since we removed the drawer
</script>

<style>
/* Adding a global primary color override */
:root {
  --q-primary: #6467F2 !important;
}
</style>
