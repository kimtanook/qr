<template>
  <QrcodeVue
    v-if="qrValue"
    :value="qrValue"
    :size="300"
    :level="level"
    :render-as="renderAs"
  />
  <input v-model="inputValue" />
  <button @click="qrValue = inputValue">create</button>
</template>
<script setup lang="ts">
import type {Level, RenderAs} from "qrcode.vue";
import QrcodeVue from "qrcode.vue";
import {ref} from "vue";

const qrValue = ref("");
const inputValue = ref("");
const level = ref<Level>("M");
const renderAs = ref<RenderAs>("svg");
</script>

<style scoped></style>
