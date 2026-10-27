<template>
  <div class="picture-upload">
    <a-upload
      list-type="picture-card"
      :show-upload-list="false"
      :custom-request="handleUpload"
      :before-upload="beforeUpload"
    >
      <img v-if="picture?.url" :src="picture?.url" alt="uploaded picture" />
      <div v-else class="upload-empty">
        <loading-outlined v-if="loading"></loading-outlined>
        <plus-outlined v-else></plus-outlined>
        <div class="ant-upload-text">点击或拖拽上传图片</div>
        <small>支持 jpg / png，建议小于 2MB</small>
      </div>
    </a-upload>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { LoadingOutlined, PlusOutlined } from '@ant-design/icons-vue'
import type { UploadProps } from 'ant-design-vue'
import { message } from 'ant-design-vue'
import { uploadPictureUsingPost } from '@/api/pictureController.ts'

interface Props {
  picture?: API.PictureVO
  spaceId?: number | string
  onSuccess?: (newPicture: API.PictureVO) => void
}

const props = defineProps<Props>()
const loading = ref<boolean>(false)

const handleUpload = async ({ file }: any) => {
  loading.value = true
  try {
    const params: API.PictureUploadRequest = props.picture ? { id: props.picture.id } : {}
    params.spaceId = props.spaceId
    const res = await uploadPictureUsingPost(params, {}, file)
    if (res.data.code === 0 && res.data.data) {
      message.success('图片上传成功')
      props.onSuccess?.(res.data.data)
    } else {
      message.error('图片上传失败：' + res.data.message)
    }
  } catch (error: any) {
    console.error('图片上传失败', error)
    message.error('图片上传失败：' + error.message)
  }
  loading.value = false
}

const beforeUpload = (file: UploadProps['fileList'][number]) => {
  const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png'
  if (!isJpgOrPng) {
    message.error('仅支持 jpg 或 png 图片')
  }
  const isLt2M = file.size / 1024 / 1024 < 2
  if (!isLt2M) {
    message.error('图片不能超过 2MB')
  }
  return isJpgOrPng && isLt2M
}
</script>

<style scoped>
.picture-upload :deep(.ant-upload) {
  width: 100% !important;
  height: 280px !important;
  min-width: 152px;
  min-height: 220px;
  border-style: dashed !important;
  background:
    radial-gradient(circle at 50% 0%, rgba(255, 157, 203, 0.22), transparent 38%),
    rgba(255, 255, 255, 0.68) !important;
}

.picture-upload img {
  max-width: 100%;
  max-height: 480px;
  border-radius: 8px;
  object-fit: contain;
}

.upload-empty {
  color: var(--xl-primary);
}

.upload-empty .ant-upload-text {
  margin-top: 10px;
  color: var(--xl-text);
  font-weight: 700;
}

.upload-empty small {
  display: block;
  margin-top: 6px;
  color: var(--xl-muted);
}
</style>
