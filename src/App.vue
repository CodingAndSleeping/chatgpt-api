<script setup lang="ts">
import { ref } from "vue";
import ChatIndex from "./components/ChatIndex.vue";
import ChatgptIndex from "./components/ChatgptIndex.vue";

let flag = ref<boolean>(false);
</script>

<template>
  <KeepAlive>
    <component :is="flag ? ChatIndex : ChatgptIndex"></component>
  </KeepAlive>

  <el-button @click="flag = !flag">切换</el-button>
</template>

<style lang="scss">
html,
body {
  height: 100%;
  width: 100%;
  padding: 0;
  margin: 0;
}
</style>
