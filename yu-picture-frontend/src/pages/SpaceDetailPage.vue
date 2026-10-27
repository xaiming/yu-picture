<template>
  <div id="spaceDetailPage" class="anime-page">
    <section class="space-hero anime-panel">
      <div>
        <p class="anime-page-subtitle">Personal Space</p>
        <h2 class="anime-page-title">{{ space.spaceName }}（{{ SPACE_TYPE_MAP[space.spaceType] }}）</h2>
      </div>
      <a-space size="middle" wrap>
        <a-button
          v-if="canUploadPicture"
          type="primary"
          :href="`/add_picture?spaceId=${id}`"
          target="_blank"
        >
          上传图片
        </a-button>
        <a-button
          v-if="canManageSpaceUser"
          type="primary"
          ghost
          :icon="h(TeamOutlined)"
          :href="`/spaceUserManage/${id}`"
          target="_blank"
        >
          成员管理
        </a-button>
        <a-button
          v-if="canManageSpaceUser"
          type="primary"
          ghost
          :icon="h(BarChartOutlined)"
          :href="`/space_analyze?spaceId=${id}`"
          target="_blank"
        >
          空间分析
        </a-button>
        <a-button v-if="canEditPicture" :icon="h(EditOutlined)" @click="doBatchEdit">
          批量编辑
        </a-button>
        <a-tooltip :title="`占用空间 ${formatSize(space.totalSize)} / ${formatSize(space.maxSize)}`">
          <a-progress
            type="circle"
            :size="42"
            :percent="Number(((space.totalSize * 100) / space.maxSize).toFixed(1))"
          />
        </a-tooltip>
      </a-space>
    </section>

    <section class="anime-panel search-panel">
      <PictureSearchForm :onSearch="onSearch" />
      <a-form-item label="按颜色搜索" class="color-search">
        <color-picker format="hex" @pureColorChange="onColorChange" />
      </a-form-item>
    </section>

    <PictureList
      :dataList="dataList"
      :loading="loading"
      :showOp="true"
      :canEdit="canEditPicture"
      :canDelete="canDeletePicture"
      :onReload="fetchData"
    />
    <a-pagination
      class="pagination"
      v-model:current="searchParams.current"
      v-model:pageSize="searchParams.pageSize"
      :total="total"
      @change="onPageChange"
    />
    <BatchEditPictureModal
      ref="batchEditPictureModalRef"
      :spaceId="id"
      :pictureList="dataList"
      :onSuccess="onBatchEditPictureSuccess"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, h, onMounted, ref, watch } from 'vue'
import { getSpaceVoByIdUsingGet } from '@/api/spaceController.ts'
import { message } from 'ant-design-vue'
import {
  listPictureVoByPageUsingPost,
  searchPictureByColorUsingPost,
} from '@/api/pictureController.ts'
import { formatSize } from '@/utils'
import PictureList from '@/components/PictureList.vue'
import PictureSearchForm from '@/components/PictureSearchForm.vue'
import { ColorPicker } from 'vue3-colorpicker'
import 'vue3-colorpicker/style.css'
import BatchEditPictureModal from '@/components/BatchEditPictureModal.vue'
import { BarChartOutlined, EditOutlined, TeamOutlined } from '@ant-design/icons-vue'
import { SPACE_PERMISSION_ENUM, SPACE_TYPE_MAP } from '../constants/space.ts'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const space = ref<API.SpaceVO>({})

function createPermissionChecker(permission: string) {
  return computed(() => (space.value.permissionList ?? []).includes(permission))
}

const canManageSpaceUser = createPermissionChecker(SPACE_PERMISSION_ENUM.SPACE_USER_MANAGE)
const canUploadPicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_UPLOAD)
const canEditPicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_EDIT)
const canDeletePicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_DELETE)

const fetchSpaceDetail = async () => {
  try {
    const res = await getSpaceVoByIdUsingGet({ id: props.id })
    if (res.data.code === 0 && res.data.data) {
      space.value = res.data.data
    } else {
      message.error('获取空间详情失败：' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取空间详情失败：' + e.message)
  }
}

const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)
const searchParams = ref<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})

const fetchData = async () => {
  loading.value = true
  const res = await listPictureVoByPageUsingPost({
    spaceId: props.id,
    ...searchParams.value,
  })
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败：' + res.data.message)
  }
  loading.value = false
}

const onPageChange = (page: number, pageSize: number) => {
  searchParams.value.current = page
  searchParams.value.pageSize = pageSize
  fetchData()
}

const onSearch = (newSearchParams: API.PictureQueryRequest) => {
  searchParams.value = {
    ...searchParams.value,
    ...newSearchParams,
    current: 1,
  }
  fetchData()
}

const onColorChange = async (color: string) => {
  loading.value = true
  const res = await searchPictureByColorUsingPost({
    picColor: color,
    spaceId: props.id,
  })
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data ?? []
    total.value = dataList.value.length
  } else {
    message.error('按颜色搜索失败：' + res.data.message)
  }
  loading.value = false
}

const batchEditPictureModalRef = ref()
const onBatchEditPictureSuccess = () => {
  fetchData()
}

const doBatchEdit = () => {
  batchEditPictureModalRef.value?.openModal()
}

watch(
  () => props.id,
  () => {
    fetchSpaceDetail()
    fetchData()
  },
)

onMounted(() => {
  fetchSpaceDetail()
  fetchData()
})
</script>

<style scoped>
#spaceDetailPage {
  display: grid;
  gap: 18px;
}

.space-hero,
.search-panel {
  padding: 22px;
}

.space-hero {
  display: flex;
  gap: 18px;
  align-items: center;
  justify-content: space-between;
}

.color-search {
  margin: 14px 0 0;
}

.pagination {
  text-align: right;
}

@media (max-width: 900px) {
  .space-hero {
    align-items: flex-start;
    flex-direction: column;
  }
}
</style>
