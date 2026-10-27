<template>
  <div id="addPictureBatchPage" class="anime-page">
    <section class="page-hero anime-panel">
      <p class="anime-page-subtitle">Batch Import</p>
      <h2 class="anime-page-title">批量抓取图片</h2>
      <p>输入关键词，让星璃云图自动收集一组灵感素材。</p>
    </section>

    <section class="form-panel anime-panel">
      <a-form name="formData" layout="vertical" :model="formData" @finish="handleSubmit">
        <a-form-item name="searchText" label="关键词">
          <a-input v-model:value="formData.searchText" placeholder="请输入关键词，如 少女、星空、头像" allow-clear />
        </a-form-item>
        <a-form-item name="count" label="抓取数量">
          <a-input-number
            v-model:value="formData.count"
            placeholder="请输入数量"
            style="min-width: 180px"
            :min="1"
            :max="30"
            allow-clear
          />
        </a-form-item>
        <a-form-item name="namePrefix" label="名称前缀">
          <a-input
            v-model:value="formData.namePrefix"
            placeholder="请输入名称前缀，会自动补充序号"
            allow-clear
          />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" html-type="submit" block :loading="loading">
            执行抓取任务
          </a-button>
        </a-form-item>
      </a-form>
    </section>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { message } from 'ant-design-vue'
import { uploadPictureByBatchUsingPost } from '@/api/pictureController.ts'
import { useRouter } from 'vue-router'

const formData = reactive<API.PictureUploadByBatchRequest>({
  count: 10,
})
const loading = ref(false)
const router = useRouter()

const handleSubmit = async () => {
  loading.value = true
  const res = await uploadPictureByBatchUsingPost({
    ...formData,
  })
  if (res.data.code === 0 && res.data.data) {
    message.success(`创建成功，共 ${res.data.data} 条`)
    router.push({ path: '/gallery' })
  } else {
    message.error('创建失败：' + res.data.message)
  }
  loading.value = false
}
</script>

<style scoped>
#addPictureBatchPage {
  display: grid;
  max-width: 780px;
  gap: 18px;
  margin: 0 auto;
}

.page-hero,
.form-panel {
  padding: 28px;
}

.page-hero {
  background:
    radial-gradient(circle at 92% 10%, rgba(255, 157, 203, 0.26), transparent 34%),
    rgba(255, 255, 255, 0.72);
}

.page-hero p:last-child {
  margin: 12px 0 0;
  color: var(--xl-muted);
}
</style>
