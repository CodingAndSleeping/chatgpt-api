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


    <el-icon class="switchBtn" @click="flag = !flag" :size="30"><Switch /></el-icon>


</template>

<style lang="scss">

@import "github-markdown-css/github-markdown-dark.css";
html,
body {
  height: 100%;
  width: 100%;
  padding: 0;
  margin: 0;
}

.switchBtn{
  position: fixed;
  top: 1vh;
  left: 0.5vw;
  border: 1px solid white;
  background-color: white;
}

</style>
