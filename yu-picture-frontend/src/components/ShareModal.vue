<template>
  <a-modal v-model:visible="visible" :title="title" :footer="false" @cancel="closeModal">
    <h4>复制分享链接</h4>
    <a-typography-link copyable>
      {{ link }}
    </a-typography-link>
    <div class="qr-title">手机扫码查看</div>
    <a-qrcode :value="link" />
  </a-modal>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

interface Props {
  title?: string
  link?: string
}

withDefaults(defineProps<Props>(), {
  title: '分享图片',
  link: window.location.origin,
})

const visible = ref(false)

const openModal = () => {
  visible.value = true
}

const closeModal = () => {
  visible.value = false
}

defineExpose({
  openModal,
})
</script>

<style scoped>
.qr-title {
  margin: 16px 0 10px;
  font-weight: 700;
}
</style>
