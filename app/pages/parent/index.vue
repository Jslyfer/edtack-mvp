<template>
  <div class="home">
    <AppHeader :isParent="true" />
    <main class="flex flex-col">
      <Children />
    </main>
    <!-- <AppFooter /> -->
  </div>
</template>

<script setup>
import Children from "~/pages/parent/Children.vue";
</script>

<style lang="scss" scoped>
</style>
