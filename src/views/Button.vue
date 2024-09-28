<template>
  <button class="btn">
    <slot></slot>
  </button>
</template>

<script></script>

<style scoped>
.btn {
  background-color: #888888;
  border-radius: 5px;
  border: none;
  padding: 10px 15px;
  box-shadow: 10px 5px 5px #888888;
  color: #ffffff;
}

.btn:hover {
  background-color: #636363;
}
</style>
