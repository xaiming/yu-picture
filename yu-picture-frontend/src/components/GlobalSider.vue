<template>
  <div id="globalSider">
    <a-layout-sider
      v-if="loginUserStore.loginUser.id"
      width="210"
      breakpoint="lg"
      collapsed-width="0"
    >
      <div class="sider-card">
        <p class="sider-label">星璃空间</p>
        <a-menu
          v-model:selectedKeys="current"
          mode="inline"
          :items="menuItems"
          @click="doMenuClick"
        />
      </div>
    </a-layout-sider>
  </div>
</template>

<script lang="ts" setup>
import { computed, h, ref, watchEffect } from 'vue'
import { CloudOutlined, PictureOutlined, TeamOutlined, UserOutlined } from '@ant-design/icons-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { SPACE_TYPE_ENUM } from '@/constants/space.ts'
import { listMyTeamSpaceUsingPost } from '@/api/spaceUserController.ts'
import { message } from 'ant-design-vue'

const loginUserStore = useLoginUserStore()

const fixedMenuItems = [
  {
    key: '/gallery',
    icon: () => h(PictureOutlined),
    label: '公共图库',
  },
  {
    key: '/my_space',
    label: '我的空间',
    icon: () => h(UserOutlined),
  },
  {
    key: '/add_space?type=' + SPACE_TYPE_ENUM.TEAM,
    label: '创建星域',
    icon: () => h(TeamOutlined),
  },
]

const teamSpaceList = ref<API.SpaceUserVO[]>([])
const menuItems = computed(() => {
  if (teamSpaceList.value.length < 1) {
    return fixedMenuItems
  }
  const teamSpaceSubMenus = teamSpaceList.value.map((spaceUser) => {
    const space = spaceUser.space
    return {
      key: '/space/' + spaceUser.spaceId,
      icon: () => h(CloudOutlined),
      label: space?.spaceName,
    }
  })
  return [
    ...fixedMenuItems,
    {
      type: 'group',
      label: '协作星域',
      key: 'teamSpace',
      children: teamSpaceSubMenus,
    },
  ]
})

const fetchTeamSpaceList = async () => {
  const res = await listMyTeamSpaceUsingPost()
  if (res.data.code === 0 && res.data.data) {
    teamSpaceList.value = res.data.data
  } else {
    message.error('加载协作星域失败：' + res.data.message)
  }
}

watchEffect(() => {
  if (loginUserStore.loginUser.id) {
    fetchTeamSpaceList()
  }
})

const router = useRouter()
const current = ref<string[]>([])

router.afterEach((to) => {
  current.value = [to.path === '/' ? '/gallery' : to.path]
})

const doMenuClick = ({ key }: { key: string }) => {
  router.push(key)
}
</script>

<style scoped>
#globalSider .ant-layout-sider {
  background: transparent;
}

.sider-card {
  min-height: calc(100vh - 160px);
  padding: 14px 10px;
  border: 1px solid rgba(124, 108, 255, 0.14);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.58);
  box-shadow: 0 18px 48px rgba(112, 110, 255, 0.12);
  backdrop-filter: blur(16px);
}

.sider-label {
  margin: 4px 12px 12px;
  color: var(--xl-muted);
  font-size: 12px;
  font-weight: 700;
}

.sider-card :deep(.ant-menu) {
  border-inline-end: 0 !important;
  background: transparent;
}

.sider-card :deep(.ant-menu-item) {
  border-radius: 8px;
}

.sider-card :deep(.ant-menu-item-selected) {
  background: linear-gradient(135deg, rgba(124, 108, 255, 0.14), rgba(255, 157, 203, 0.18));
  color: var(--xl-primary);
}
</style>
