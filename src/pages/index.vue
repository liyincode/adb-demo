<script setup lang="ts">
import AdbWebUsbBackend, { AdbWebUsbBackendWatcher } from '@yume-chan/adb-backend-webusb'
import type { AdbBackend, AdbPacketData, AdbPacketInit } from '@yume-chan/adb'
// 当前浏览器是否支持
const supported = ref(false)
// 获取到的 adb 设备列表
const backendList = ref<AdbBackend[]>()
// 当前选择的 adb 设备
const selectedBackend = ref<AdbBackend>()
// 是否正在连接
const connecting = ref(false)

function init() {
  supported.value = AdbWebUsbBackend.isSupported()
  if (!supported.value) {
    console.error('Your browser does not support WebUSB standard, which is required for this site to work.\n\nLatest version of Google Chrome, Microsoft Edge, or other Chromium-based browsers are required.')
    return
  }

  updateUsbBackendList()

  const watcher = new AdbWebUsbBackendWatcher(async (serial?: string) => {
    const list = await updateUsbBackendList()

    if (serial)
      selectedBackend.value = list.find(backend => backend.serial === serial)
  })

  // onUnmounted(() => watcher.dispose())
}

async function updateUsbBackendList() {
  const _backendList = await AdbWebUsbBackend.getDevices()
  console.log('list', _backendList)

  backendList.value = _backendList

  return _backendList
}

const addUsbBackend = async () => {
  const backend = await AdbWebUsbBackend.requestDevice()
  selectedBackend.value = backend
}

const connect = async () => {
  if (!selectedBackend.value)
    return

  connecting.value = true

  let readable: ReadableStream<AdbPacketData>
  let writable: WritableStream<AdbPacketInit>
  try {
    const streams = await selectedBackend.value.connect()

    readable = streams.readable
  }
  catch (e: any) {
    connecting.value = false
  }
}

init()
</script>

<template>
  <div>
    <p>{{ supported ? '当前浏览器支持' : '当前浏览器不支持' }}</p>
    <template v-if="backendList">
      <div v-for="item in backendList" :key="item.serial">
        {{ item }}
      </div>
    </template>

    <div>
      <button
        class="m-3 text-sm btn"
        @click="addUsbBackend"
      >
        添加设备
      </button>
      <button
        class="m-3 text-sm btn bg-sky-600"
        :disabled="connecting"
        @click="connect"
      >
        {{ connecting ? '正在连接请稍后' : '连接设备' }}
      </button>
    </div>
  </div>
</template>
