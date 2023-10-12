<script setup>
import MessageComponent from "@/components/MessageComponent.vue";
import ChartComponent from "@/components/ChartComponent.vue";
import StatusComponent from "@/components/StatusComponent.vue";
</script>

<template>
  <v-app>
    <v-app-bar class="" title="Log hut"></v-app-bar>
    <v-main>
      <StatusComponent />
      <ChartComponent />
      <MessageComponent />
    </v-main>
  </v-app>
</template>

<style scoped>
</style>