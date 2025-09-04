<script setup>
import { ref } from "vue";

// Reactive state using ref
const count = ref(0);

// Method to increment the count
const increment = () => {
  count.value++;
};

// Method to decrement the count
const decrement = () => {
  count.value--;
};
</script>

<template>
  <div>
    <h1>Counter App</h1>
    <p>Current Count: {{ count }}</p>
    <button @click="decrement">-</button>
    <button @click="increment">+</button>
  </div>
</template>

<style scoped>
div {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-top: 50px;
}

button {
  margin: 10px;
  padding: 8px 15px;
  font-size: 16px;
  cursor: pointer;
}
</style>
