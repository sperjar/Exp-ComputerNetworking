<!-- eslint-disable no-alert -->
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

function crcEncoding(rawData: string, polyData: string) {
  const checkNum = polyData.length - 1 // length of check digit (CRC)
  const polyBitsArray = polyData.split('').map(x => parseInt(x))
  const data = rawData.split('').map(x => parseInt(x)).concat(new Array(checkNum).fill(0))
  const bitsXOR = (a: number[], b: number[]) => a.map((x, i) => x ^ b[i])
  for (let i = 0; i < rawData.length; i++) {
    if (data[i] === 0)
      continue
    data.splice(i, polyData.length, ...bitsXOR(data.slice(i, i + polyData.length), polyBitsArray))
  }
  return rawData + data.slice(-checkNum).join('')
}

function crcValidating(data: string, polyData: string): boolean {
  const checkNum = polyData.length - 1 // length of check digit (CRC)
  const polyBitsArray = polyData.split('').map(x => parseInt(x))
  const bitsXOR = (a: number[], b: number[]) => a.map((x, i) => x ^ b[i])
  const dataBitsArray = data.split('').map(x => parseInt(x))
  for (let i = 0; i < dataBitsArray.length - checkNum; i++) {
    if (dataBitsArray[i] === 0)
      continue
    dataBitsArray.splice(i, polyData.length, ...bitsXOR(dataBitsArray.slice(i, i + polyData.length), polyBitsArray))
  }
  return dataBitsArray.slice(-polyData.length).every(x => x === 0)
}

function crcDecoding(data: string, polyData: string): string {
  return data.slice(0, -polyData.length + 1)
}

function encode(): void {
  // check input
  if (rawData.value.length === 0) {
    alert('请输入原始数据')
    throw new Error('原始数据输入有误：无原始数据')
  }
  else if (!rawData.value.match(/^[01]+$/)) {
    alert('原始数据输入有误：请输入仅包含`0`和`1`的字符串')
    throw new Error('原始数据输入有误：原始数据格式错误')
  }
  else if (rawPolyData.value.length === 0) {
    alert('请输入生成多项式')
    throw new Error('生成多项式输入有误：无生成多项式')
  }
  else if (!rawPolyData.value.match(/1[0|1]*1|1/)) {
    alert('生成多项式输入有误：请输入仅包含`0`和`1`的字符串，且第一位和最后一位必须为`1`')
    throw new Error('生成多项式输入有误：生成多项式格式错误')
  }

  encodingData.value = crcEncoding(rawData.value, rawPolyData.value)
}

function validate(): void {
  // check input
  if (encodingData.value.length === 0) {
    validResult.value = ValidResult.Empty
    alert('请输入编码后的数据')
    throw new Error('编码后的数据输入有误：无编码后的数据')
  }
  else if (!encodingData.value.match(/^[01]+$/)) {
    validResult.value = ValidResult.Empty
    alert('编码后的数据输入有误：请输入仅包含`0`和`1`的字符串')
    throw new Error('编码后的数据输入有误：编码后的数据格式错误')
  }
  else if (rawPolyData.value.length === 0) {
    validResult.value = ValidResult.Empty
    alert('请输入生成多项式')
    throw new Error('生成多项式输入有误：无生成多项式')
  }
  else if (!rawPolyData.value.match(/1[0|1]*1|1/)) {
    validResult.value = ValidResult.Empty
    alert('生成多项式输入有误：请输入仅包含`0`和`1`的字符串，且第一位和最后一位必须为`1`')
    throw new Error('生成多项式输入有误：生成多项式格式错误')
  }

  if (crcValidating(encodingData.value, rawPolyData.value))
    validResult.value = ValidResult.Valid
  else
    validResult.value = ValidResult.Invalid
}

function decode(): void {
  validate()
  decodingData.value = crcDecoding(encodingData.value, rawPolyData.value)
}
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
      <input id="poly_dta" v-model="rawPolyData" class="value" type="text" pattern="1[0|1]*1|1">
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
    <button @click="encode()">
      编码
    </button>
    <button @click="validate()">
      校验
    </button>
    <button @click="decode()">
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
