<script setup lang="ts">
// Minimal blank layout — no header, no drawer, no wayfinder.
</script>

<template>
  <div class="blank-shell">
    <slot />
  </div>
</template>

<style scoped>
.blank-shell {
  position: relative;
  width: 100%;
  height: 100%;
  background: #000;
  overflow: hidden;
}
</style>
