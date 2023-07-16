<script setup lang="ts">
import { ref } from "vue";
import Editor from "./components/Editor.vue";

const content = ref("");
</script>

<template>
  <v-container>
    <Editor v-model="content" :max-limit="280" />
  </v-container>

  <v-container>
    <code>
      {{ content }}
    </code>
  </v-container>
</template>

<style lang="scss"></style>
