<script setup lang="ts">
import { RouterView } from 'vue-router'
import ConfirmDialog from 'primevue/confirmdialog'
import Toast from 'primevue/toast';

</script>

<template>
    <ConfirmDialog />
    <Toast />
  <RouterView />
</template>

<style scoped>
</style>
