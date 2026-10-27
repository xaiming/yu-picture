<template>
  <main class="showcase-page">
    <div class="dream-light"></div>
    <div class="petals petals-a"></div>
    <div class="petals petals-b"></div>
    <section class="showcase-stage">
      <div class="copy">
        <p>观赏模式</p>
        <h1>星璃云图</h1>
        <span>在柔光、花瓣与插画展台之间，预览你的二次元灵感收藏</span>
      </div>

      <div class="orbit-scene" aria-label="环形旋转图库">
        <div class="orbit-glow"></div>
        <div class="orbit-ring">
          <button
            v-for="(picture, index) in displayPictures"
            :key="picture.id ?? index"
            class="orbit-card"
            :style="getCardStyle(index)"
            type="button"
            @click="openPicture(picture)"
          >
            <img
              v-if="picture.url"
              :src="picture.thumbnailUrl ?? picture.url"
              :alt="picture.name ?? 'gallery picture'"
            />
            <span v-else>{{ picture.name }}</span>
          </button>
        </div>
      </div>

      <img class="assistant" :src="animeAssistant" alt="星璃虚拟助手" />
      <RouterLink class="enter-button" to="/gallery">进入图库</RouterLink>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import { listPictureVoByPageUsingPost } from '@/api/pictureController'
import animeAssistant from '@/assets/anime-assistant.png'

type ShowcasePicture = Pick<API.PictureVO, 'id' | 'name' | 'url' | 'thumbnailUrl'>

const router = useRouter()
const pictures = ref<ShowcasePicture[]>([])

const fallbackPictures: ShowcasePicture[] = [
  { id: 1, name: '星空', url: '' },
  { id: 2, name: '少女', url: '' },
  { id: 3, name: '梦幻', url: '' },
  { id: 4, name: '蓝紫', url: '' },
  { id: 5, name: '治愈', url: '' },
  { id: 6, name: '未来', url: '' },
  { id: 7, name: '插画', url: '' },
  { id: 8, name: '灵感', url: '' },
]

const displayPictures = computed(() => {
  const source = pictures.value.length > 0 ? pictures.value : fallbackPictures
  return source.slice(0, 10)
})

const getCardStyle = (index: number) => {
  const count = displayPictures.value.length || 1
  return {
    '--angle': `${(360 / count) * index}deg`,
  }
}

const openPicture = (picture: ShowcasePicture) => {
  if (!picture.id || !picture.url) {
    return
  }
  router.push(`/picture/${picture.id}`)
}

const fetchShowcasePictures = async () => {
  try {
    const res = await listPictureVoByPageUsingPost({
      current: 1,
      pageSize: 10,
      sortField: 'createTime',
      sortOrder: 'descend',
    })
    if (res.data.code === 0 && res.data.data?.records) {
      pictures.value = res.data.data.records.filter((picture) => picture.url)
    }
  } catch (error) {
    console.warn('load showcase pictures failed', error)
  }
}

onMounted(() => {
  fetchShowcasePictures()
})
</script>

<style scoped>
.showcase-page {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  background:
    radial-gradient(circle at 12% 20%, rgba(255, 186, 218, 0.46), transparent 28%),
    radial-gradient(circle at 82% 12%, rgba(154, 212, 255, 0.5), transparent 30%),
    radial-gradient(circle at 52% 88%, rgba(183, 170, 255, 0.42), transparent 34%),
    linear-gradient(135deg, #fbfdff 0%, #f1ecff 48%, #fff2fa 100%);
  color: #27243d;
}

.dream-light {
  position: absolute;
  inset: -20%;
  background:
    conic-gradient(from 110deg, transparent, rgba(124, 108, 255, 0.2), transparent, rgba(255, 157, 203, 0.25), transparent),
    linear-gradient(120deg, transparent, rgba(255, 255, 255, 0.72), transparent);
  filter: blur(24px);
  animation: dream-drift 18s ease-in-out infinite alternate;
}

.petals {
  position: absolute;
  inset: -10%;
  background-image:
    radial-gradient(ellipse at center, rgba(255, 157, 203, 0.58) 0 34%, transparent 36%),
    radial-gradient(ellipse at center, rgba(124, 108, 255, 0.22) 0 32%, transparent 34%),
    radial-gradient(circle, rgba(255, 255, 255, 0.72) 0 24%, transparent 26%);
  background-size: 120px 120px, 170px 170px, 82px 82px;
  opacity: 0.5;
}

.petals-a {
  animation: petal-float 60s linear infinite;
}

.petals-b {
  opacity: 0.28;
  animation: petal-float 92s linear infinite reverse;
}

.showcase-stage {
  position: relative;
  z-index: 1;
  min-height: 100vh;
  display: grid;
  place-items: center;
  perspective: 1200px;
}

.copy {
  position: absolute;
  top: clamp(36px, 7vh, 80px);
  width: min(90vw, 760px);
  text-align: center;
}

.copy p {
  margin: 0 0 10px;
  color: rgba(255, 255, 255, 0.72);
  color: #7c6cff;
  font-weight: 800;
}

.copy h1 {
  margin: 0;
  font-size: clamp(48px, 9vw, 108px);
  line-height: 0.95;
  letter-spacing: 0;
  text-shadow: 0 18px 52px rgba(124, 108, 255, 0.2);
}

.copy span {
  display: block;
  margin-top: 14px;
  color: rgba(72, 68, 112, 0.72);
  font-size: 16px;
}

.orbit-scene {
  position: relative;
  width: min(86vw, 840px);
  height: min(68vw, 520px);
  transform-style: preserve-3d;
}

.orbit-glow {
  position: absolute;
  inset: 17% 21%;
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.16), transparent 62%);
  box-shadow: 0 0 80px rgba(124, 108, 255, 0.2);
}

.orbit-ring {
  position: absolute;
  inset: 0;
  transform-style: preserve-3d;
  animation: orbit-spin 30s linear infinite;
}

.orbit-card {
  position: absolute;
  top: 50%;
  left: 50%;
  width: clamp(112px, 15vw, 174px);
  aspect-ratio: 4 / 5;
  padding: 0;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.28);
  border-radius: 8px;
  background:
    radial-gradient(circle at 30% 18%, rgba(255, 255, 255, 0.42), transparent 24%),
    linear-gradient(135deg, rgba(124, 108, 255, 0.82), rgba(255, 157, 203, 0.72));
  box-shadow: 0 24px 60px rgba(124, 108, 255, 0.22);
  color: #fff;
  cursor: pointer;
  transform:
    translate(-50%, -50%)
    rotateY(var(--angle))
    translateZ(clamp(230px, 34vw, 420px));
  transform-style: preserve-3d;
}

.orbit-card img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.orbit-card span {
  display: grid;
  width: 100%;
  height: 100%;
  place-items: center;
  font-size: 22px;
  font-weight: 800;
}

.assistant {
  position: absolute;
  right: min(4vw, 46px);
  bottom: 0;
  width: min(28vw, 340px);
  max-height: 72vh;
  object-fit: contain;
  filter: drop-shadow(0 24px 44px rgba(124, 108, 255, 0.28));
  pointer-events: none;
}

.enter-button {
  position: absolute;
  bottom: clamp(44px, 8vh, 84px);
  display: inline-flex;
  min-width: 150px;
  height: 46px;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(124, 108, 255, 0.22);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.72);
  box-shadow: 0 18px 44px rgba(124, 108, 255, 0.22);
  color: #6f5cff;
  font-weight: 800;
  backdrop-filter: blur(16px);
  opacity: 0;
  transform: translateY(18px);
  animation: enter-fade 0.9s ease 3s forwards;
}

.enter-button:hover {
  color: #fff;
  background: linear-gradient(135deg, #7c6cff, #ff9dcb);
}

@keyframes orbit-spin {
  from {
    transform: rotateX(-8deg) rotateY(0deg);
  }
  to {
    transform: rotateX(-8deg) rotateY(-360deg);
  }
}

@keyframes dream-drift {
  from {
    transform: translate(-4%, -2%) rotate(0deg);
  }
  to {
    transform: translate(4%, 2%) rotate(18deg);
  }
}

@keyframes petal-float {
  from {
    transform: translateY(0) rotate(0deg);
  }
  to {
    transform: translateY(8%) rotate(360deg);
  }
}

@keyframes enter-fade {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 980px) {
  .assistant {
    opacity: 0.24;
    width: 42vw;
  }
}

@media (max-width: 720px) {
  .orbit-scene {
    width: 100vw;
    height: 58vh;
  }

  .assistant {
    display: none;
  }

  .orbit-card {
    transform:
      translate(-50%, -50%)
      rotateY(var(--angle))
      translateZ(clamp(160px, 42vw, 260px));
  }
}
</style>
