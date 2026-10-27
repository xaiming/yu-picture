<template>
  <div class="picture-list">
    <a-list
      :grid="{ gutter: 18, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
      :data-source="dataList"
      :loading="loading"
    >
      <template #renderItem="{ item: picture }">
        <a-list-item class="picture-item">
          <a-card class="picture-card" hoverable @click="doClickPicture(picture)">
            <template #cover>
              <div class="cover-wrap">
                <img
                  v-if="picture.thumbnailUrl || picture.url"
                  :alt="picture.name"
                  :src="picture.thumbnailUrl ?? picture.url"
                />
                <div v-else class="cover-fallback">星璃</div>
              </div>
            </template>
            <a-card-meta :title="picture.name || '未命名作品'">
              <template #description>
                <a-flex wrap="wrap" gap="small">
                  <a-tag color="purple">
                    {{ picture.category ?? '灵感' }}
                  </a-tag>
                  <a-tag v-for="tag in picture.tags ?? []" :key="tag">
                    {{ tag }}
                  </a-tag>
                </a-flex>
              </template>
            </a-card-meta>
            <template v-if="showOp" #actions>
              <ShareAltOutlined @click="(e) => doShare(picture, e)" />
              <SearchOutlined @click="(e) => doSearch(picture, e)" />
              <EditOutlined v-if="canEdit" @click="(e) => doEdit(picture, e)" />
              <DeleteOutlined v-if="canDelete" @click="(e) => doDelete(picture, e)" />
            </template>
          </a-card>
        </a-list-item>
      </template>
    </a-list>
    <ShareModal ref="shareModalRef" :link="shareLink" />
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import {
  DeleteOutlined,
  EditOutlined,
  SearchOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'
import { deletePictureUsingPost } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import ShareModal from '@/components/ShareModal.vue'
import { ref } from 'vue'

interface Props {
  dataList?: API.PictureVO[]
  loading?: boolean
  showOp?: boolean
  canEdit?: boolean
  canDelete?: boolean
  onReload?: () => void
}

const props = withDefaults(defineProps<Props>(), {
  dataList: () => [],
  loading: false,
  showOp: false,
  canEdit: false,
  canDelete: false,
})

const router = useRouter()

const doClickPicture = (picture: API.PictureVO) => {
  router.push({
    path: `/picture/${picture.id}`,
  })
}

const doSearch = (picture: API.PictureVO, e: MouseEvent) => {
  e.stopPropagation()
  window.open(`/search_picture?pictureId=${picture.id}`)
}

const doEdit = (picture: API.PictureVO, e: MouseEvent) => {
  e.stopPropagation()
  router.push({
    path: '/add_picture',
    query: {
      id: picture.id,
      spaceId: picture.spaceId,
    },
  })
}

const doDelete = async (picture: API.PictureVO, e: MouseEvent) => {
  e.stopPropagation()
  const id = picture.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    props.onReload?.()
  } else {
    message.error('删除失败')
  }
}

const shareModalRef = ref()
const shareLink = ref<string>()

const doShare = (picture: API.PictureVO, e: MouseEvent) => {
  e.stopPropagation()
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.id}`
  shareModalRef.value?.openModal()
}
</script>

<style scoped>
.picture-item {
  padding: 0 !important;
}

.picture-card {
  overflow: hidden;
  border-radius: 8px;
  transition:
    transform 0.22s ease,
    box-shadow 0.22s ease;
}

.picture-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 24px 52px rgba(124, 108, 255, 0.2);
}

.cover-wrap {
  position: relative;
  width: 100%;
  aspect-ratio: 4 / 3;
  overflow: hidden;
  background:
    radial-gradient(circle at 30% 20%, rgba(255, 157, 203, 0.45), transparent 36%),
    linear-gradient(135deg, #ebe7ff, #e5f7ff);
}

.cover-wrap img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.24s ease;
}

.picture-card:hover .cover-wrap img {
  transform: scale(1.05);
}

.cover-fallback {
  display: grid;
  width: 100%;
  height: 100%;
  place-items: center;
  color: #8b77ff;
  font-size: 24px;
  font-weight: 800;
}

.picture-card :deep(.ant-card-body) {
  padding: 13px;
}

.picture-card :deep(.ant-card-meta-title) {
  color: var(--xl-text);
  font-weight: 800;
}
</style>
