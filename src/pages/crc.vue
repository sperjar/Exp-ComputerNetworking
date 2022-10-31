<script lang="ts" setup>
enum ValidResult {
  Valid,
  Invalid,
  Empty,
}

const validResult = ref(ValidResult.Empty)
const rawData = ref('')
const rawPolyData = ref('')
const encodingData = ref('')
const decodingData = ref('')
</script>

<template>
  <h1 text-3xl mb-12 font-bold>
    CRC 编码
  </h1>
  <div my-8>
    <div class="item">
      <label for="raw_data">原始数据</label>
      <input id="raw_data" v-model="rawData" class="value" type="text" pattern="[0|1]+">
    </div>
    <div class="item">
      <label for="poly_data">生成多项式</label>
      <input id="poly_dta" v-model="rawPolyData" class="value" type="text">
    </div>
    <div class="item">
      <label for="encoding_data">编码数据</label>
      <input id="encoding_data" v-model="encodingData" class="value" type="text" pattern="[0|1]+">
    </div>
    <div class="item">
      <label for="decoding_data">解码数据</label>
      <input id="decoding_data" v-model="decodingData" class="value" type="text" pattern="[0|1]+">
    </div>
    <div class="item">
      <div v-if="validResult === ValidResult.Empty" mx-auto text-2xl bg-gray-400 i-carbon-unknown />
      <div v-if="validResult === ValidResult.Invalid" mx-auto text-2xl bg-red-600 i-carbon-close />
      <div v-if="validResult === ValidResult.Valid" mx-auto text-2xl bg-teal-600 i-carbon-checkmark />
    </div>
  </div>
  <div my-8>
    <button>
      编码
    </button>
    <button>
      校验
    </button>
    <button>
      解码
    </button>
  </div>
</template>

<style scoped>
.item {
  @apply my-6;
}

label {
  @apply inline-block w-24 text-left;
}

.value {
  @apply inline-block w-48;
}

.validity {
  @apply text-left w-12;
}

input {
  @apply px-1 w-48 inline-block;
  @apply outline-none border-2 border-gray-200 dark-border-gray-700 rounded;
  @apply bg-transparent;
  @apply focus-border-teal-600;
}

input:invalid {
  @apply border-red-600;
}

button {
  @apply btn mx-2 text-sm;
}
</style>
