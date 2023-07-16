<script setup lang="ts">
import { ref } from "vue";
import Editor from "./components/Editor.vue";

const content = ref("");
</script>

<template>
  <v-container id="app">
    <Editor v-model="content" :max-limit="280" />
  </v-container>
</template>

<style lang="scss"></style>
