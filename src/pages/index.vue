<script setup lang="ts">
import AdbWebUsbBackend, { AdbWebUsbBackendWatcher } from '@yume-chan/adb-backend-webusb'
import type { AdbBackend } from '@yume-chan/adb'
// 当前浏览器是否支持
const supported = ref(false)
// 获取到的 adb 设备列表
const backendList = ref<AdbBackend[]>()
// 当前选择的 adb 设备
const selectedBackend = ref<AdbBackend>()

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

  onUnmounted(() => watcher.dispose())
}

async function updateUsbBackendList() {
  const _backendList = await AdbWebUsbBackend.getDevices()
  backendList.value = _backendList

  return _backendList
}

init()

const go = () => {

}
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
        @click="go"
      >
        Go
      </button>
    </div>
  </div>
</template>
