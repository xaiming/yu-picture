<template>
  <div id="searchPicturePage" class="anime-page">
    <section class="page-hero anime-panel">
      <p class="anime-page-subtitle">Image Search</p>
      <h2 class="anime-page-title">以图搜图</h2>
      <p>根据当前作品寻找相似灵感和来源参考。</p>
    </section>

    <section class="origin-panel anime-panel">
      <h3>原图</h3>
      <a-card hoverable class="origin-card">
        <template #cover>
          <img :alt="picture.name" :src="picture.thumbnailUrl ?? picture.url" />
        </template>
        <a-card-meta :title="picture.name || '待识别图片'" />
      </a-card>
    </section>

    <section class="result-panel">
      <h3>识图结果</h3>
      <a-list
        :grid="{ gutter: 18, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
        :data-source="dataList"
        :loading="loading"
      >
        <template #renderItem="{ item: result }">
          <a-list-item class="result-item">
            <a :href="result.fromUrl" target="_blank">
              <a-card hoverable class="result-card">
                <template #cover>
                  <img :alt="result.name" :src="result.thumbUrl" />
                </template>
                <a-card-meta :title="result.name || '相似图片'" />
              </a-card>
            </a>
          </a-list-item>
        </template>
      </a-list>
    </section>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import {
  getPictureVoByIdUsingGet,
  searchPictureByPictureUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const pictureId = computed(() => route.query?.pictureId)
const picture = ref<API.PictureVO>({})
const dataList = ref<API.ImageSearchResult[]>([])
const loading = ref<boolean>(true)

const fetchPictureDetail = async () => {
  try {
    const res = await getPictureVoByIdUsingGet({ id: pictureId.value })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      message.error('获取图片详情失败：' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取图片详情失败：' + e.message)
  }
}

const fetchResultData = async () => {
  loading.value = true
  try {
    const res = await searchPictureByPictureUsingPost({ pictureId: pictureId.value })
    if (res.data.code === 0 && res.data.data) {
      dataList.value = res.data.data ?? []
    } else {
      message.error('获取识图结果失败：' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取识图结果失败：' + e.message)
  }
  loading.value = false
}

onMounted(() => {
  fetchPictureDetail()
  fetchResultData()
})
</script>

<style scoped>
#searchPicturePage {
  display: grid;
  gap: 18px;
}

.page-hero,
.origin-panel {
  padding: 24px;
}

.page-hero p:last-child {
  margin: 12px 0 0;
  color: var(--xl-muted);
}

.origin-panel h3,
.result-panel h3 {
  margin: 0 0 14px;
  color: var(--xl-text);
}

.origin-card {
  width: 260px;
}

.origin-card img,
.result-card img {
  width: 100%;
  height: 180px;
  object-fit: cover;
}

.result-item {
  padding: 0 !important;
}
</style>
