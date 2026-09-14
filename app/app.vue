<template>
  <NuxtRouteAnnouncer />
  <NuxtPage />
</template>

<script setup lang="ts">
useHead({
  script: [
    {
      src: 'https://accounts.google.com/gsi/client',
      async: true,
      defer: true,
    },
  ],
})
</script>

<style>
.brand-mark {
  display: none !important;
}
</style>