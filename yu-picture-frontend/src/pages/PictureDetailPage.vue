<template>
  <div id="pictureDetailPage" class="anime-page">
    <a-row :gutter="[18, 18]">
      <a-col :sm="24" :md="16" :xl="18">
        <a-card title="图片预览">
          <div class="preview-wrap">
            <a-image :src="picture.url" style="max-height: 640px; object-fit: contain" />
          </div>
        </a-card>
      </a-col>
      <a-col :sm="24" :md="8" :xl="6">
        <a-card title="作品信息">
          <a-descriptions :column="1">
            <a-descriptions-item label="作者">
              <a-space>
                <a-avatar :size="24" :src="picture.user?.userAvatar" />
                <div>{{ picture.user?.userName ?? '星璃用户' }}</div>
              </a-space>
            </a-descriptions-item>
            <a-descriptions-item label="名称">
              {{ picture.name ?? '未命名作品' }}
            </a-descriptions-item>
            <a-descriptions-item label="简介">
              {{ picture.introduction ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="分类">
              {{ picture.category ?? '灵感' }}
            </a-descriptions-item>
            <a-descriptions-item label="标签">
              <a-tag v-for="tag in picture.tags" :key="tag">
                {{ tag }}
              </a-tag>
            </a-descriptions-item>
            <a-descriptions-item label="格式">
              {{ picture.picFormat ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="宽度">
              {{ picture.picWidth ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="高度">
              {{ picture.picHeight ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="宽高比">
              {{ picture.picScale ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="大小">
              {{ formatSize(picture.picSize) }}
            </a-descriptions-item>
            <a-descriptions-item label="主色调">
              <a-space>
                {{ picture.picColor ?? '-' }}
                <div
                  v-if="picture.picColor"
                  class="color-chip"
                  :style="{ backgroundColor: toHexColor(picture.picColor) }"
                />
              </a-space>
            </a-descriptions-item>
          </a-descriptions>
          <a-space wrap class="detail-actions">
            <a-button type="primary" @click="doDownload">
              免费下载
              <template #icon>
                <DownloadOutlined />
              </template>
            </a-button>
            <a-button :icon="h(ShareAltOutlined)" type="primary" ghost @click="doShare">
              分享
            </a-button>
            <a-button v-if="canEdit" :icon="h(EditOutlined)" @click="doEdit">
              编辑
            </a-button>
            <a-button v-if="canDelete" :icon="h(DeleteOutlined)" danger @click="doDelete">
              删除
            </a-button>
          </a-space>
        </a-card>
      </a-col>
    </a-row>
    <ShareModal ref="shareModalRef" :link="shareLink" />
  </div>
</template>

<script setup lang="ts">
import { computed, h, onMounted, ref } from 'vue'
import { deletePictureUsingPost, getPictureVoByIdUsingGet } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import {
  DeleteOutlined,
  DownloadOutlined,
  EditOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'
import { useRouter } from 'vue-router'
import { downloadImage, formatSize, toHexColor } from '@/utils'
import ShareModal from '@/components/ShareModal.vue'
import { SPACE_PERMISSION_ENUM } from '@/constants/space.ts'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const picture = ref<API.PictureVO>({})
const router = useRouter()

function createPermissionChecker(permission: string) {
  return computed(() => (picture.value.permissionList ?? []).includes(permission))
}

const canEdit = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_EDIT)
const canDelete = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_DELETE)

const fetchPictureDetail = async () => {
  try {
    const res = await getPictureVoByIdUsingGet({ id: props.id })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      message.error('获取图片详情失败：' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取图片详情失败：' + e.message)
  }
}

const doEdit = () => {
  router.push({
    path: '/add_picture',
    query: {
      id: picture.value.id,
      spaceId: picture.value.spaceId,
    },
  })
}

const doDelete = async () => {
  const id = picture.value.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
  } else {
    message.error('删除失败')
  }
}

const doDownload = () => {
  downloadImage(picture.value.url)
}

const shareModalRef = ref()
const shareLink = ref<string>()

const doShare = () => {
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.value.id}`
  shareModalRef.value?.openModal()
}

onMounted(() => {
  fetchPictureDetail()
})
</script>

<style scoped>
.preview-wrap {
  display: grid;
  min-height: 420px;
  place-items: center;
  border-radius: 8px;
  background:
    radial-gradient(circle at 30% 10%, rgba(255, 157, 203, 0.18), transparent 32%),
    rgba(255, 255, 255, 0.46);
}

.detail-actions {
  margin-top: 18px;
}

.color-chip {
  width: 16px;
  height: 16px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.8);
}
</style>
