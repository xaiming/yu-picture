<template>
  <div id="mySpacePage" class="anime-page">
    <section class="loading-card anime-panel">
      <p class="anime-page-subtitle">My Space</p>
      <h2 class="anime-page-title">正在进入我的空间</h2>
      <p>星璃正在为你寻找个人图库空间...</p>
      <a-spin />
    </section>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { listSpaceVoByPageUsingPost } from '@/api/spaceController.ts'
import { message } from 'ant-design-vue'
import { onMounted } from 'vue'
import { SPACE_TYPE_ENUM } from '@/constants/space.ts'

const router = useRouter()
const loginUserStore = useLoginUserStore()

const checkUserSpace = async () => {
  const loginUser = loginUserStore.loginUser
  if (!loginUser?.id) {
    router.replace('/user/login')
    return
  }
  const res = await listSpaceVoByPageUsingPost({
    userId: loginUser.id,
    current: 1,
    pageSize: 1,
    spaceType: SPACE_TYPE_ENUM.PRIVATE,
  })
  if (res.data.code === 0) {
    if (res.data.data?.records?.length > 0) {
      const space = res.data.data.records[0]
      router.replace(`/space/${space.id}`)
    } else {
      router.replace('/add_space')
      message.warn('请先创建空间')
    }
  } else {
    message.error('加载我的空间失败：' + res.data.message)
  }
}

onMounted(() => {
  checkUserSpace()
})
</script>

<style scoped>
#mySpacePage {
  display: grid;
  min-height: calc(100vh - 220px);
  place-items: center;
}

.loading-card {
  width: min(100%, 520px);
  padding: 32px;
  text-align: center;
}

.loading-card p {
  color: var(--xl-muted);
}
</style>
