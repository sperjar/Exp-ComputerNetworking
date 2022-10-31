<!-- eslint-disable no-alert -->
<script lang="ts" setup>
enum ValidResult {
  Valid,
  Invalid,
  Empty,
}

const validResult = ref(ValidResult.Empty)
const rawData = ref('')
const encodingData = ref('')
const decodingData = ref('')

function checkInput(data: string): boolean {
  return /^[01]+$/.test(data)
}

function hammingEncoding(_rawData: string): string {
  // check input
  if (_rawData.length === 0) {
    alert('请输入原始数据')
    throw new Error('原始数据输入有误：无原始数据')
  }
  else if (!checkInput(_rawData)) {
    alert('原始数据输入有误：请输入仅包含`0`和`1`的字符串')
    throw new Error('原始数据输入有误：原始数据格式错误')
  }

  // fix k, r, n
  // 2^r >= k + r + 1
  // k: data length
  // r: parity bit length
  // n: total length
  const k = _rawData.length
  let r = 0
  for (let i = 0; ; i++) {
    if (2 ** i >= k + i + 1) {
      r = i
      break
    }
  }
  const n = k + r

  // fill data bits
  const code: Array<number> = new Array(n).fill(0)
  const isParityBit = (i: number): boolean => i === 1 || (i & (i - 1)) === 0

  for (let i = 0, j = 0; i < n; i++) {
    const parityCnt = Math.floor(Math.log2(i + 1))
    if (isParityBit(i + 1) && parityCnt < r)
      continue
    code[i] = parseInt(_rawData[j++])
  }

  // calculate parity bits

  // judge whether the bit is to be calculated
  // index starts from 1
  // ri starts from 1, stands for the i-th parity bit
  const genCalBit = (ri: number) => {
    const list: Array<number> = []
    for (let j = 0; ; j++) {
      const left = (1 + 2 * j) * 2 ** (ri - 1)
      let right = (j + 1) * 2 ** ri - 1
      if (left <= n) {
        right = Math.min(right, n)
        for (let i = left; i <= right; i++)
          list.push(i)
      }
      else {
        break
      }
    }
    return list
  }

  for (let i = 0; i < r; i++) {
    const calBit = genCalBit(i + 1)
    let sum = 0
    for (const bit of calBit)
      sum += code[bit - 1]
    code[2 ** i - 1] = sum % 2
  }

  return code.join('')
}

function hammingValidating(_decodingData: string): number {
  // check input
  if (_decodingData.length === 0) {
    alert('请输入编码数据')
    throw new Error('编码数据输入有误：无编码数据')
  }
  else if (!checkInput(_decodingData)) {
    alert('编码数据输入有误：请输入仅包含`0`和`1`的字符串')
    throw new Error('编码数据输入有误：编码数据格式错误')
  }

  // fix k, r, n
  // 2^r >= k + r + 1
  // k: data length
  // r: parity bit length
  // n: total length
  const n = _decodingData.length
  const r = Math.ceil(Math.log2(n + 1))

  // calculate parity bits
  // judge whether the bit is to be calculated
  // index starts from 1
  // ri starts from 1, stands for the i-th parity bit
  const genCalBit = (ri: number) => {
    const list: Array<number> = []
    for (let j = 0; ; j++) {
      const left = (1 + 2 * j) * 2 ** (ri - 1)
      let right = (j + 1) * 2 ** ri - 1
      if (left <= n) {
        right = Math.min(right, n)
        for (let i = left; i <= right; i++)
          list.push(i)
      }
      else {
        break
      }
    }
    return list
  }

  const validateCode: Array<number> = []
  for (let i = 0; i < r; i++) {
    const calBit = genCalBit(i + 1)
    let sum = 0
    for (const bit of calBit)
      sum += parseInt(_decodingData[bit - 1])
    validateCode.push(sum % 2)
  }

  // check error
  const errorCode = parseInt(validateCode.reverse().join(''), 2)
  return errorCode
}

function hammingDecoding(_encodingData: string): string {
  const isParityBit = (i: number): boolean => i === 1 || (i & (i - 1)) === 0
  const data = _encodingData.split('').map(x => parseInt(x)).filter((_, i) => !isParityBit(i + 1))
  return data.join('')
}

function encode(): void {
  encodingData.value = hammingEncoding(rawData.value)
}

function decode(): void {
  const errorCode = hammingValidating(encodingData.value)
  if (errorCode === 0) {
    validResult.value = ValidResult.Valid
    decodingData.value = hammingDecoding(encodingData.value)
  }
  else {
    validResult.value = ValidResult.Invalid
    const errorBit = errorCode - 1
    const data = encodingData.value.split('').map(x => parseInt(x))
    data[errorBit] = (data[errorBit] + 1) % 2
    decodingData.value = hammingDecoding(data.join(''))
  }
}

function validate(): void {
  const errorCode = hammingValidating(encodingData.value)
  if (errorCode !== 0) {
    validResult.value = ValidResult.Invalid
    alert(`校验失败，错误位为${errorCode}`)
    throw new Error(`校验失败，错误位为${errorCode}`)
  }
  else {
    validResult.value = ValidResult.Valid
  }
}
</script>

<template>
  <h1 text-3xl mb-12 font-bold>
    海明编码
  </h1>
  <div my-8>
    <div class="item">
      <label for="raw_data">原始数据</label>
      <input id="raw_data" v-model="rawData" class="value" type="text" pattern="[0|1]+">
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
