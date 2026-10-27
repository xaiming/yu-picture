<template>
  <main id="homePage">
    <section class="gallery-shell">
      <aside class="sidebar">
        <RouterLink class="brand" to="/gallery">
          <span class="brand-orbit">✦</span>
          <span class="brand-text">星璃云图</span>
        </RouterLink>

        <nav class="side-nav">
          <RouterLink
            v-for="item in menuItems"
            :key="item.label"
            class="side-item"
            :class="{ active: item.active }"
            :to="item.to"
          >
            <component :is="item.icon" />
            <span>{{ item.label }}</span>
          </RouterLink>
        </nav>

        <RouterLink class="watch-button" to="/">
          <StarOutlined />
          <span>观赏模式</span>
          <CaretRightOutlined />
        </RouterLink>

        <div class="storage-card">
          <CloudOutlined />
          <div>
            <strong>存储空间</strong>
            <span>1.2 TB / 2 TB</span>
          </div>
          <div class="storage-bar">
            <i></i>
          </div>
        </div>
      </aside>

      <section class="main-panel">
        <header class="topbar">
          <label class="search-box">
            <SearchOutlined />
            <input
              v-model="searchText"
              placeholder="搜索图片、标签、分类..."
              @keyup.enter="doSearch"
            />
          </label>
          <div class="top-actions">
            <button type="button" aria-label="通知">
              <BellOutlined />
            </button>
            <button type="button" aria-label="消息">
              <MailOutlined />
            </button>
            <button type="button" aria-label="收藏">
              <StarOutlined />
            </button>
            <div class="profile">
              <img :src="animeGirl" alt="星璃酱头像" />
              <div>
                <strong>星璃酱</strong>
                <span>一位热爱收藏的小画师</span>
              </div>
            </div>
          </div>
        </header>

        <section class="content-scroll">
          <section class="welcome-card">
            <div class="welcome-copy">
              <h1>欢迎回来，星璃酱 ✨</h1>
              <p>在 星璃云图 的第 128 天，遇见 {{ totalText }} 张美好瞬间</p>
            </div>
            <div class="floating-castle"></div>
          </section>

          <section class="stats-grid">
            <article v-for="stat in stats" :key="stat.label" class="stat-card">
              <span class="stat-icon">
                <component :is="stat.icon" />
              </span>
              <div>
                <small>{{ stat.label }}</small>
                <strong>{{ stat.value }}</strong>
              </div>
            </article>
          </section>

          <section class="toolbar">
            <div class="category-list">
              <button
                v-for="category in categories"
                :key="category"
                type="button"
                :class="{ active: selectedCategory === category }"
                @click="selectCategory(category)"
              >
                {{ category }}
              </button>
            </div>
            <div class="toolbar-actions">
              <button class="mode-button active" type="button" aria-label="九宫格视图">
                <AppstoreOutlined />
              </button>
              <button class="mode-button" type="button" aria-label="列表视图">
                <UnorderedListOutlined />
              </button>
              <RouterLink class="upload-button" to="/add_picture">
                <CloudUploadOutlined />
                上传图片
              </RouterLink>
            </div>
          </section>

          <section class="gallery-area">
            <div class="cards-grid">
              <article
                v-for="picture in displayPictures"
                :key="picture.key"
                class="picture-card"
                @click="openPicture(picture)"
              >
                <div
                  class="cover"
                  :class="{ 'cover-has-image': !!picture.url }"
                  :style="picture.url ? undefined : { background: picture.cover }"
                >
                  <img v-if="picture.url" :src="picture.thumbnailUrl || picture.url" :alt="picture.title" />
                  <span>{{ picture.badge }}</span>
                </div>
                <div class="card-body">
                  <div class="card-title-row">
                    <h3>{{ picture.title }}</h3>
                    <button type="button" aria-label="更多" @click.stop>
                      <MoreOutlined />
                    </button>
                  </div>
                  <div class="card-tags">
                    <span>{{ picture.category }}</span>
                    <em>{{ picture.tag }}</em>
                  </div>
                  <div class="card-meta">
                    <span><HeartOutlined /> {{ picture.likes }}</span>
                    <span><EyeOutlined /> {{ picture.views }}</span>
                  </div>
                </div>
              </article>
            </div>

            <aside class="right-deck">
              <img class="anime-girl" :src="animeGirl" alt="二次元少女助手" />
              <div class="mini-panel mood-panel">
                <span>今日氛围</span>
                <strong>86%</strong>
                <p>灵感指数稳定上升</p>
              </div>
              <div class="mini-panel quick-panel">
                <span>快捷操作</span>
                <a href="/add_picture">新建上传</a>
                <a href="/add_picture/batch">批量抓取</a>
                <a href="/my_space">我的空间</a>
              </div>
            </aside>
          </section>
        </section>
      </section>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import animeGirl from '@/assets/anime-assistant.png'
import { listPictureVoByPageUsingPost } from '@/api/pictureController.ts'
import {
  AppstoreOutlined,
  BellOutlined,
  CaretRightOutlined,
  CloudOutlined,
  CloudUploadOutlined,
  DeleteOutlined,
  EyeOutlined,
  FolderOutlined,
  HeartOutlined,
  HomeOutlined,
  MailOutlined,
  MoreOutlined,
  PictureOutlined,
  SearchOutlined,
  SettingOutlined,
  StarOutlined,
  TagsOutlined,
  TeamOutlined,
  UnorderedListOutlined,
} from '@ant-design/icons-vue'

type DisplayPicture = {
  key: string
  id?: string | number
  title: string
  category: string
  tag: string
  likes: string
  views: string
  badge: string
  cover: string
  url?: string
  thumbnailUrl?: string
}

const router = useRouter()
const searchText = ref('')
const selectedCategory = ref('全部')
const realPictures = ref<API.PictureVO[]>([])
const total = ref(0)

const menuItems = [
  { label: '公共图库', to: '/gallery', icon: HomeOutlined, active: true },
  { label: '我的空间', to: '/my_space', icon: FolderOutlined },
  { label: '星域空间', to: '/add_space', icon: TeamOutlined },
  { label: '收藏夹', to: '/gallery', icon: StarOutlined },
  { label: '回收站', to: '/gallery', icon: DeleteOutlined },
  { label: '图片管理', to: '/admin/pictureManage', icon: PictureOutlined },
  { label: '分类管理', to: '/admin/pictureManage', icon: AppstoreOutlined },
  { label: '标签管理', to: '/admin/pictureManage', icon: TagsOutlined },
  { label: '空间管理', to: '/admin/spaceManage', icon: CloudOutlined },
  { label: '系统设置', to: '/gallery', icon: SettingOutlined },
]

const categories = ['全部', '插画', '壁纸', '头像', '表情包', '角色设定', '场景背景', '灵感素材']

const fallbackPictures: DisplayPicture[] = [
  {
    key: 'mock-1',
    title: '星空下的约定',
    category: '插画',
    tag: '#少女',
    likes: '1.2k',
    views: '2.8k',
    badge: 'ILLUST',
    cover: 'linear-gradient(135deg, #7c6cff 0%, #9ad4ff 48%, #ffb5d6 100%)',
  },
  {
    key: 'mock-2',
    title: '云端樱花站',
    category: '场景背景',
    tag: '#治愈',
    likes: '824',
    views: '1.5k',
    badge: 'SCENE',
    cover: 'linear-gradient(135deg, #ffb7dc 0%, #d6c6ff 48%, #9ad4ff 100%)',
  },
  {
    key: 'mock-3',
    title: '电子天使',
    category: '角色设定',
    tag: '#科技感',
    likes: '1.7k',
    views: '3.2k',
    badge: 'CHARA',
    cover: 'linear-gradient(135deg, #5c73ff 0%, #a28bff 50%, #ffe0f1 100%)',
  },
  {
    key: 'mock-4',
    title: '甜心女孩',
    category: '头像',
    tag: '#可爱',
    likes: '1.1k',
    views: '2.0k',
    badge: 'AVATAR',
    cover: 'linear-gradient(135deg, #ffe7f4 0%, #ff9dcb 45%, #a99cff 100%)',
  },
]

const totalText = computed(() => total.value || 3472)

const stats = computed(() => [
  { label: '图片总数', value: String(totalText.value), icon: PictureOutlined },
  { label: '分类数量', value: '28', icon: FolderOutlined },
  { label: '标签数量', value: '86', icon: TagsOutlined },
  { label: '浏览量', value: '18.6w', icon: EyeOutlined },
])

const displayPictures = computed<DisplayPicture[]>(() => {
  if (!realPictures.value.length) {
    return fallbackPictures
  }
  return realPictures.value.slice(0, 8).map((picture, index) => {
    const tags = picture.tags ?? []
    return {
      key: String(picture.id ?? index),
      id: picture.id,
      title: picture.name || `星璃作品 ${index + 1}`,
      category: picture.category || '灵感',
      tag: tags.length > 0 ? `#${tags[0]}` : '#二次元',
      likes: `${Math.max(1, index + 1)}.${(index * 3 + 2) % 10}k`,
      views: `${Math.max(2, index + 2)}.${(index * 5 + 8) % 10}k`,
      badge: picture.picFormat?.toUpperCase() || 'IMAGE',
      cover: fallbackPictures[index % fallbackPictures.length].cover,
      url: picture.url,
      thumbnailUrl: picture.thumbnailUrl,
    }
  })
})

const fetchPictures = async () => {
  const params: API.PictureQueryRequest = {
    current: 1,
    pageSize: 8,
    sortField: 'createTime',
    sortOrder: 'descend',
    searchText: searchText.value || undefined,
  }
  if (selectedCategory.value !== '全部') {
    params.category = selectedCategory.value
  }
  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data) {
    realPictures.value = res.data.data.records ?? []
    total.value = Number(res.data.data.total ?? 0)
  }
}

const doSearch = () => {
  fetchPictures()
}

const selectCategory = (category: string) => {
  selectedCategory.value = category
  fetchPictures()
}

const openPicture = (picture: DisplayPicture) => {
  if (picture.id) {
    router.push(`/picture/${picture.id}`)
  }
}

onMounted(() => {
  fetchPictures()
})
</script>

<style scoped>
#homePage {
  position: fixed;
  inset: 0;
  z-index: 99;
  overflow: auto;
  padding: 22px;
  background:
    radial-gradient(circle at 88% 10%, rgba(255, 160, 217, 0.42), transparent 30%),
    radial-gradient(circle at 10% 90%, rgba(133, 178, 255, 0.36), transparent 34%),
    linear-gradient(135deg, #f4f6ff 0%, #edf1ff 50%, #fff2fb 100%);
  color: #29305b;
  font-family: "Microsoft YaHei", "PingFang SC", sans-serif;
}

.gallery-shell {
  display: grid;
  grid-template-columns: 240px minmax(0, 1fr);
  width: min(1660px, calc(100vw - 44px));
  min-height: calc(100vh - 44px);
  margin: 0 auto;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.72);
  border-radius: 28px;
  background: rgba(255, 255, 255, 0.42);
  box-shadow: 0 36px 90px rgba(83, 88, 177, 0.22);
  backdrop-filter: blur(24px);
}

.sidebar {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 18px;
  padding: 24px 20px;
  overflow: hidden;
  background:
    radial-gradient(circle at 32% 6%, rgba(255, 255, 255, 0.32), transparent 30%),
    linear-gradient(180deg, #254cbd 0%, #6656d8 52%, #8ab8ff 100%);
  color: #fff;
}

.sidebar::before,
.welcome-card::before {
  position: absolute;
  inset: 0;
  content: "";
  background-image:
    radial-gradient(rgba(255, 255, 255, 0.95) 1px, transparent 1px),
    radial-gradient(rgba(255, 255, 255, 0.55) 1px, transparent 1px);
  background-size: 64px 64px, 110px 110px;
  opacity: 0.34;
  pointer-events: none;
}

.brand {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  gap: 12px;
  min-height: 54px;
  color: #fff;
}

.brand-orbit {
  display: grid;
  width: 48px;
  height: 48px;
  place-items: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #fff, #a8dfff 45%, #e7c2ff);
  box-shadow: 0 0 28px rgba(255, 255, 255, 0.78);
  color: #6656d8;
  font-size: 25px;
}

.brand-text {
  font-size: 24px;
  font-weight: 900;
  letter-spacing: 0;
  text-shadow: 0 4px 18px rgba(255, 255, 255, 0.26);
}

.side-nav {
  position: relative;
  z-index: 1;
  display: grid;
  gap: 8px;
}

.side-item {
  display: flex;
  height: 48px;
  align-items: center;
  gap: 12px;
  padding: 0 16px;
  border: 1px solid transparent;
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.86);
  font-size: 15px;
  font-weight: 700;
}

.side-item svg {
  font-size: 18px;
}

.side-item.active,
.side-item:hover {
  border-color: rgba(255, 255, 255, 0.64);
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.3), rgba(255, 170, 224, 0.42));
  box-shadow: inset 0 0 18px rgba(255, 255, 255, 0.2), 0 10px 24px rgba(60, 58, 171, 0.2);
  color: #fff;
}

.watch-button {
  position: relative;
  z-index: 1;
  display: flex;
  height: 58px;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-top: auto;
  border: 1px solid rgba(255, 255, 255, 0.64);
  border-radius: 999px;
  background: linear-gradient(135deg, rgba(140, 113, 255, 0.92), rgba(255, 157, 203, 0.82));
  box-shadow: 0 18px 36px rgba(42, 50, 155, 0.28);
  color: #fff;
  font-size: 16px;
  font-weight: 900;
}

.storage-card {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 34px 1fr;
  gap: 10px;
  padding: 14px;
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.18);
  color: #fff;
}

.storage-card strong,
.storage-card span {
  display: block;
}

.storage-card span {
  margin-top: 4px;
  font-size: 12px;
  opacity: 0.76;
}

.storage-bar {
  grid-column: 1 / -1;
  height: 6px;
  overflow: hidden;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.25);
}

.storage-bar i {
  display: block;
  width: 62%;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #fff, #ffb2dc);
}

.main-panel {
  min-width: 0;
  background:
    radial-gradient(circle at 74% 8%, rgba(156, 213, 255, 0.34), transparent 28%),
    linear-gradient(135deg, rgba(255, 255, 255, 0.72), rgba(236, 241, 255, 0.74));
}

.topbar {
  display: grid;
  grid-template-columns: minmax(280px, 420px) 1fr;
  gap: 18px;
  align-items: center;
  padding: 22px 28px 14px;
}

.search-box {
  display: flex;
  height: 48px;
  align-items: center;
  gap: 12px;
  padding: 0 18px;
  border: 1px solid rgba(111, 92, 255, 0.2);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.42);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.72);
}

.search-box input {
  width: 100%;
  border: 0;
  outline: 0;
  background: transparent;
  color: #32375f;
  font-size: 14px;
}

.search-box input::placeholder {
  color: #8d91bc;
}

.top-actions {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 14px;
}

.top-actions button,
.mode-button,
.card-title-row button {
  display: grid;
  border: 0;
  place-items: center;
  background: transparent;
  color: #5c65a7;
  cursor: pointer;
}

.top-actions button {
  width: 36px;
  height: 36px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.48);
}

.profile {
  display: flex;
  min-width: 220px;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.46);
}

.profile img {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  object-fit: cover;
}

.profile strong,
.profile span {
  display: block;
}

.profile strong {
  color: #404b91;
}

.profile span {
  margin-top: 2px;
  color: #7f86b4;
  font-size: 12px;
}

.content-scroll {
  position: relative;
  height: calc(100vh - 94px);
  overflow: auto;
  padding: 0 28px 30px;
}

.welcome-card {
  position: relative;
  min-height: 148px;
  overflow: hidden;
  border-radius: 24px;
  background:
    radial-gradient(circle at 80% 30%, rgba(255, 191, 224, 0.46), transparent 28%),
    radial-gradient(circle at 58% 0%, rgba(138, 206, 255, 0.42), transparent 34%),
    linear-gradient(135deg, rgba(122, 118, 255, 0.78), rgba(186, 187, 255, 0.58), rgba(255, 255, 255, 0.34));
  box-shadow: 0 24px 60px rgba(107, 96, 209, 0.22);
}

.welcome-copy {
  position: relative;
  z-index: 1;
  padding: 30px;
}

.welcome-copy h1 {
  margin: 0;
  color: #fff;
  font-size: 28px;
  letter-spacing: 0;
  text-shadow: 0 8px 28px rgba(55, 61, 142, 0.25);
}

.welcome-copy p {
  margin: 12px 0 0;
  color: rgba(255, 255, 255, 0.86);
  font-weight: 700;
}

.floating-castle {
  position: absolute;
  right: 60px;
  bottom: -24px;
  width: 260px;
  height: 130px;
  border-radius: 999px 999px 20px 20px;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.58), rgba(255, 255, 255, 0.04)),
    linear-gradient(135deg, rgba(255, 255, 255, 0.22), rgba(116, 105, 255, 0.28));
  filter: blur(0.2px);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
  margin: 18px 0;
}

.stat-card {
  display: flex;
  align-items: center;
  gap: 14px;
  min-height: 82px;
  padding: 16px;
  border: 1px solid rgba(255, 255, 255, 0.7);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.68);
  box-shadow: 0 16px 34px rgba(112, 110, 198, 0.12);
  backdrop-filter: blur(14px);
}

.stat-icon {
  display: grid;
  width: 46px;
  height: 46px;
  place-items: center;
  border-radius: 16px;
  background: linear-gradient(135deg, rgba(124, 108, 255, 0.95), rgba(154, 212, 255, 0.8));
  color: #fff;
  font-size: 20px;
}

.stat-card small,
.stat-card strong {
  display: block;
}

.stat-card small {
  color: #7e85b2;
  font-weight: 700;
}

.stat-card strong {
  margin-top: 5px;
  color: #273477;
  font-size: 24px;
}

.toolbar {
  display: flex;
  gap: 14px;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.category-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.category-list button {
  height: 38px;
  padding: 0 18px;
  border: 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.68);
  box-shadow: 0 8px 20px rgba(107, 96, 209, 0.08);
  color: #59609c;
  font-weight: 800;
  cursor: pointer;
}

.category-list button.active {
  background: linear-gradient(135deg, #7c6cff, #ff9dcb);
  color: #fff;
}

.toolbar-actions {
  display: flex;
  flex-shrink: 0;
  gap: 10px;
  align-items: center;
}

.mode-button {
  width: 38px;
  height: 38px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.72);
}

.mode-button.active {
  color: #6b60e1;
}

.upload-button {
  display: inline-flex;
  height: 42px;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 0 22px;
  border-radius: 999px;
  background: linear-gradient(135deg, #5f7dff, #ff9dcb);
  box-shadow: 0 16px 30px rgba(108, 98, 255, 0.24);
  color: #fff;
  font-weight: 900;
}

.gallery-area {
  position: relative;
  display: grid;
  grid-template-columns: minmax(0, 1fr) 260px;
  gap: 16px;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 16px;
}

.picture-card {
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.78);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.74);
  box-shadow: 0 16px 34px rgba(75, 84, 172, 0.12);
  cursor: pointer;
  transition: transform 0.22s ease, box-shadow 0.22s ease;
}

.picture-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 26px 48px rgba(75, 84, 172, 0.22);
}

.cover {
  position: relative;
  aspect-ratio: 16 / 10;
  overflow: hidden;
}

.cover::before {
  position: absolute;
  inset: 0;
  z-index: 1;
  content: "";
  background:
    radial-gradient(circle at 38% 22%, rgba(255, 255, 255, 0.28), transparent 20%),
    radial-gradient(circle at 68% 64%, rgba(255, 255, 255, 0.18), transparent 28%);
}

.cover img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.24s ease;
}

.picture-card:hover .cover img {
  transform: scale(1.05);
}

.cover span {
  position: absolute;
  right: 12px;
  bottom: 10px;
  z-index: 2;
  padding: 5px 10px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.72);
  color: #625adc;
  font-size: 11px;
  font-weight: 900;
}

.card-body {
  padding: 12px 14px 14px;
}

.card-title-row {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: space-between;
}

.card-title-row h3 {
  margin: 0;
  color: #30376f;
  font-size: 15px;
}

.card-tags {
  display: flex;
  gap: 8px;
  margin-top: 8px;
  color: #7b81ad;
  font-size: 12px;
}

.card-tags span {
  padding: 3px 8px;
  border-radius: 999px;
  background: rgba(124, 108, 255, 0.12);
  color: #6b60e1;
  font-weight: 800;
}

.card-tags em {
  font-style: normal;
}

.card-meta {
  display: flex;
  gap: 14px;
  margin-top: 12px;
  color: #6d73a4;
  font-size: 12px;
}

.card-meta span {
  display: inline-flex;
  gap: 5px;
  align-items: center;
}

.right-deck {
  position: relative;
  min-height: 560px;
}

.anime-girl {
  position: absolute;
  right: -42px;
  bottom: 0;
  z-index: 2;
  width: 290px;
  height: 650px;
  object-fit: contain;
  object-position: bottom right;
  filter: drop-shadow(0 26px 46px rgba(91, 84, 178, 0.28));
  pointer-events: none;
}

.mini-panel {
  position: relative;
  z-index: 3;
  width: 190px;
  margin-left: 8px;
  padding: 16px;
  border: 1px solid rgba(255, 255, 255, 0.72);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.66);
  box-shadow: 0 16px 34px rgba(75, 84, 172, 0.12);
  backdrop-filter: blur(14px);
}

.mini-panel span,
.mini-panel a {
  display: block;
}

.mood-panel {
  margin-top: 22px;
}

.mood-panel strong {
  display: block;
  margin-top: 12px;
  color: #6b60e1;
  font-size: 34px;
}

.mood-panel p {
  margin: 6px 0 0;
  color: #7b81ad;
}

.quick-panel {
  margin-top: 14px;
}

.quick-panel span {
  margin-bottom: 10px;
  color: #30376f;
  font-weight: 900;
}

.quick-panel a {
  padding: 7px 0;
  color: #6269a3;
  font-weight: 700;
}

@media (max-width: 1380px) {
  .gallery-area {
    grid-template-columns: 1fr;
  }

  .right-deck {
    display: none;
  }
}

@media (max-width: 1180px) {
  .cards-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

@media (max-width: 960px) {
  #homePage {
    padding: 0;
  }

  .gallery-shell {
    grid-template-columns: 1fr;
    width: 100%;
    min-height: 100vh;
    border-radius: 0;
  }

  .sidebar {
    display: none;
  }

  .topbar {
    grid-template-columns: 1fr;
  }

  .top-actions {
    justify-content: space-between;
  }

  .stats-grid,
  .cards-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .toolbar {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (max-width: 640px) {
  .content-scroll {
    padding: 0 16px 24px;
  }

  .stats-grid,
  .cards-grid {
    grid-template-columns: 1fr;
  }

  .profile {
    min-width: 0;
  }
}
</style>
