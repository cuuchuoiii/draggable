<template>
  <div>
    <h1>{{ message }}</h1>
    <button @click="updateMessage">Update Message</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const message = ref('Hello, World!');

function updateMessage() {
  message.value = 'Hello, Vue 3!';
}
</script>
