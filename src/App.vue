<template>
  <v-container id="app">
    <Editor v-model="content" :max-limit="280" />
  </v-container>
</template>

<script setup lang="ts">
import { ref } from "vue";
import Editor from "./components/Editor.vue";

const content = ref("Hallo");
</script>

<style lang="scss" scoped>
#app {
  font-family: Arial;
  font-size: 16px;

  .submit-button {
    border: 1px solid #333;
    border-radius: 2px;
    background: #fff;
    color: #333;
    font-size: 16px;
    padding: 6px 12px;
    margin-top: 8px;
    -webkit-appearance: none;
    cursor: pointer;

    &:hover {
      background: #333;
      color: #fff;
    }
  }
}
</style>
