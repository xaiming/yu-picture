<template>
  <div id="globalHeader">
    <router-link class="brand" to="/gallery">
      <span class="brand-mark">星</span>
      <span>
        <strong>星璃云图</strong>
        <small>Anime Cloud Gallery</small>
      </span>
    </router-link>
    <a-menu
      v-model:selectedKeys="current"
      class="nav-menu"
      mode="horizontal"
      :items="items"
      @click="doMenuClick"
    />
    <div class="user-login-status">
      <div v-if="loginUserStore.loginUser.id">
        <a-dropdown>
          <a-space class="user-chip">
            <a-avatar :src="loginUserStore.loginUser.userAvatar" />
            {{ loginUserStore.loginUser.userName ?? '星璃访客' }}
          </a-space>
          <template #overlay>
            <a-menu>
              <a-menu-item>
                <router-link to="/my_space">
                  <UserOutlined />
                  我的空间
                </router-link>
              </a-menu-item>
              <a-menu-item @click="doLogout">
                <LogoutOutlined />
                退出登录
              </a-menu-item>
            </a-menu>
          </template>
        </a-dropdown>
      </div>
      <a-button v-else type="primary" href="/user/login">登录</a-button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, h, ref } from 'vue'
import {
  CloudUploadOutlined,
  HomeOutlined,
  LogoutOutlined,
  PictureOutlined,
  SettingOutlined,
  UserOutlined,
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import type { MenuProps } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { userLogoutUsingPost } from '@/api/userController.ts'

const loginUserStore = useLoginUserStore()

const originItems: MenuProps['items'] = [
  {
    key: '/gallery',
    icon: () => h(HomeOutlined),
    label: '公共图库',
    title: '公共图库',
  },
  {
    key: '/add_picture',
    icon: () => h(CloudUploadOutlined),
    label: '上传图片',
    title: '上传图片',
  },
  {
    key: '/admin/pictureManage',
    icon: () => h(PictureOutlined),
    label: '图片管理',
    title: '图片管理',
  },
  {
    key: '/admin/spaceManage',
    icon: () => h(SettingOutlined),
    label: '空间管理',
    title: '空间管理',
  },
  {
    key: '/admin/userManage',
    icon: () => h(UserOutlined),
    label: '用户管理',
    title: '用户管理',
  },
]

const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    if (String(menu?.key).startsWith('/admin')) {
      const loginUser = loginUserStore.loginUser
      return !!loginUser && loginUser.userRole === 'admin'
    }
    return true
  })
}

const items = computed(() => filterMenus(originItems))
const router = useRouter()
const current = ref<string[]>([])

router.afterEach((to) => {
  current.value = [to.path === '/' ? '/gallery' : to.path]
})

const doMenuClick = ({ key }: { key: string }) => {
  router.push({ path: key })
}

const doLogout = async () => {
  const res = await userLogoutUsingPost()
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}
</script>

<style scoped>
#globalHeader {
  display: grid;
  grid-template-columns: 220px minmax(0, 1fr) auto;
  gap: 16px;
  align-items: center;
  height: 100%;
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  color: var(--xl-text);
}

.brand-mark {
  display: grid;
  width: 42px;
  height: 42px;
  place-items: center;
  border-radius: 8px;
  background: linear-gradient(135deg, #7c6cff, #68c8ff 54%, #ff9dcb);
  box-shadow: 0 12px 28px rgba(124, 108, 255, 0.28);
  color: #fff;
  font-weight: 800;
}

.brand strong,
.brand small {
  display: block;
  line-height: 1.15;
}

.brand strong {
  font-size: 18px;
}

.brand small {
  margin-top: 4px;
  color: var(--xl-muted);
  font-size: 11px;
}

.nav-menu {
  min-width: 0;
  border-bottom: 0;
  background: transparent;
}

.user-login-status {
  display: flex;
  justify-content: flex-end;
}

.user-chip {
  padding: 6px 10px;
  border: 1px solid rgba(124, 108, 255, 0.14);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.62);
}

@media (max-width: 900px) {
  #globalHeader {
    grid-template-columns: 1fr auto;
  }

  .nav-menu {
    display: none;
  }
}
</style>
