<template>
  <div id="vipExchangePage" class="anime-page">
    <section class="vip-card anime-panel">
      <p class="anime-page-subtitle">Starlight Pass</p>
      <h2 class="anime-page-title">会员码兑换</h2>
      <p class="desc">输入兑换码，解锁更多星璃云图能力。</p>
      <a-form name="formData" layout="vertical" :model="formData" @finish="handleSubmit">
        <a-form-item name="vipCode" label="兑换码">
          <a-input v-model:value="formData.vipCode" placeholder="请输入会员兑换码" allow-clear />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" html-type="submit" block :loading="loading">
            立即兑换
          </a-button>
        </a-form-item>
      </a-form>
    </section>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { message } from 'ant-design-vue'
import { exchangeVipUsingPost } from '@/api/userController.ts'
import { useRouter } from 'vue-router'

const formData = reactive<API.VipExchangeRequest>({
  vipCode: '',
})
const loading = ref(false)
const router = useRouter()

const handleSubmit = async () => {
  if (!formData.vipCode) {
    message.error('请输入兑换码')
    return
  }
  loading.value = true
  try {
    const res = await exchangeVipUsingPost({ vipCode: formData.vipCode })
    if (res.data.code === 0 && res.data.data) {
      message.success('兑换成功')
      router.push({ path: '/gallery' })
    } else {
      message.error('兑换失败：' + res.data.message)
    }
  } catch {
    message.error('兑换失败，请稍后重试')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
#vipExchangePage {
  display: grid;
  min-height: calc(100vh - 220px);
  place-items: center;
}

.vip-card {
  width: min(100%, 520px);
  padding: 32px;
  background:
    radial-gradient(circle at 95% 0%, rgba(255, 157, 203, 0.3), transparent 34%),
    rgba(255, 255, 255, 0.76);
}

.desc {
  margin: 12px 0 24px;
  color: var(--xl-muted);
}
</style>
