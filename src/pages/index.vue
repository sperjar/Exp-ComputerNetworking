<script setup lang="ts">
</script>

<template>
  <h1 text-3xl font-bold mb-12>
    计算机网络实验代码
  </h1>
  <ol>
    <li>
      <router-link to="/crc">
        实验二：1. CRC 编码
      </router-link>
      <span i-carbon-link />
    </li>
    <li>
      <router-link to="/hamming">
        实验二：2. 海明编码
      </router-link>
      <span i-carbon-link />
    </li>
  </ol>
</template>

<style scoped>
a {
  @apply hover-text-teal-600 underline;
}

ul {
  @apply my-8;
}

li {
  @apply my-4;
}

span {
  @apply inline-block align-middle mx-1;
}
</style>
