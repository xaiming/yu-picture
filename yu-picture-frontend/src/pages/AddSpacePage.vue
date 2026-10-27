<template>
  <div id="addSpacePage" class="anime-page">
    <section class="anime-panel page-card">
      <p class="anime-page-subtitle">Space Studio</p>
      <h2 class="anime-page-title">
        {{ route.query?.id ? '编辑' : '创建' }} {{ SPACE_TYPE_MAP[spaceType] }}
      </h2>
      <a-form class="space-form" name="spaceForm" layout="vertical" :model="spaceForm" @finish="handleSubmit">
        <a-form-item name="spaceName" label="空间名称">
          <a-input v-model:value="spaceForm.spaceName" placeholder="请输入空间名称" allow-clear />
        </a-form-item>
        <a-form-item name="spaceLevel" label="空间级别">
          <a-select
            v-model:value="spaceForm.spaceLevel"
            style="min-width: 180px"
            placeholder="请选择空间级别"
            :options="SPACE_LEVEL_OPTIONS"
            allow-clear
          />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" html-type="submit" :loading="loading" block>
            保存星域
          </a-button>
        </a-form-item>
      </a-form>
    </section>

    <a-card class="level-card" title="空间级别说明">
      <a-typography-paragraph>
        当前支持基础空间级别，可根据容量和图片数量规划你的个人图库或协作星域。
      </a-typography-paragraph>
      <a-typography-paragraph v-for="spaceLevel in spaceLevelList" :key="spaceLevel.value">
        {{ spaceLevel.text }}：大小 {{ formatSize(spaceLevel.maxSize) }}，数量
        {{ spaceLevel.maxCount }}
      </a-typography-paragraph>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { message } from 'ant-design-vue'
import {
  addSpaceUsingPost,
  getSpaceVoByIdUsingGet,
  listSpaceLevelUsingGet,
  updateSpaceUsingPost,
} from '@/api/spaceController.ts'
import { useRoute, useRouter } from 'vue-router'
import { SPACE_LEVEL_OPTIONS, SPACE_TYPE_ENUM, SPACE_TYPE_MAP } from '@/constants/space.ts'
import { formatSize } from '../utils'

const space = ref<API.SpaceVO>()
const spaceForm = reactive<API.SpaceAddRequest | API.SpaceEditRequest>({})
const loading = ref(false)
const route = useRoute()
const router = useRouter()

const spaceType = computed(() => {
  if (route.query?.type) {
    return Number(route.query.type)
  }
  return SPACE_TYPE_ENUM.PRIVATE
})

const spaceLevelList = ref<API.SpaceLevel[]>([])

const fetchSpaceLevelList = async () => {
  const res = await listSpaceLevelUsingGet()
  if (res.data.code === 0 && res.data.data) {
    spaceLevelList.value = res.data.data
  } else {
    message.error('获取空间级别失败：' + res.data.message)
  }
}

const handleSubmit = async () => {
  const spaceId = space.value?.id
  loading.value = true
  const res = spaceId
    ? await updateSpaceUsingPost({
        id: spaceId,
        ...spaceForm,
      })
    : await addSpaceUsingPost({
        ...spaceForm,
        spaceType: spaceType.value,
      })
  if (res.data.code === 0 && res.data.data) {
    message.success('保存成功')
    router.push({
      path: `/space/${res.data.data}`,
    })
  } else {
    message.error('保存失败：' + res.data.message)
  }
  loading.value = false
}

const getOldSpace = async () => {
  const id = route.query?.id
  if (id) {
    const res = await getSpaceVoByIdUsingGet({ id })
    if (res.data.code === 0 && res.data.data) {
      const data = res.data.data
      space.value = data
      spaceForm.spaceName = data.spaceName
      spaceForm.spaceLevel = data.spaceLevel
    }
  }
}

onMounted(() => {
  fetchSpaceLevelList()
  getOldSpace()
})
</script>

<style scoped>
#addSpacePage {
  display: grid;
  max-width: 760px;
  gap: 18px;
  margin: 0 auto;
}

.page-card {
  padding: 28px;
}

.space-form {
  margin-top: 24px;
}
</style>
