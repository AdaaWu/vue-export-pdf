<script setup>
import HelloWorld from './components/DemoItem.vue'
</script>

<template>
  <div>

  <HelloWorld msg="Vite + Vue" />
  </div>

</template>

<style scoped>
h1{
  color: aliceblue;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
