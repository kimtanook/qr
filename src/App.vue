<template>
  <div>
    <button @click="tab = 'create'">Create</button>
    <button @click="tab = 'reader'">Reader</button>
  </div>
  <QrCreator v-if="tab === 'create'" />
  <QrReader v-else />
</template>
<script setup lang="ts">
import {ref} from "vue";
import QrCreator from "./components/QrCreator.vue";
import QrReader from "./components/QrReader.vue";
const tab = ref("create");
</script>

<style scoped></style>
