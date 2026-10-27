<template>
  <div id="spaceAnalyzePage" class="anime-page">
    <section class="analyze-hero anime-panel">
      <p class="anime-page-subtitle">Data Observatory</p>
      <h2 class="anime-page-title">
        图库数据分析 -
        <span v-if="queryAll">全部空间</span>
        <span v-else-if="queryPublic">公共图库</span>
        <span v-else>
          <a :href="`/space/${spaceId}`" target="_blank">空间 {{ spaceId }}</a>
        </span>
      </h2>
    </section>
    <a-row :gutter="[18, 18]">
      <a-col :xs="24" :md="12">
        <SpaceUsageAnalyze :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
      <a-col :xs="24" :md="12">
        <SpaceCategoryAnalyze :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
      <a-col :xs="24" :md="12">
        <SpaceTagAnalyze :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
      <a-col :xs="24" :md="12">
        <SpaceSizeAnalyze :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
      <a-col :xs="24" :md="12">
        <SpaceUserAnalyze :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
      <a-col :xs="24" :md="12">
        <SpaceRankAnalyze v-if="isAdmin" :spaceId="spaceId" :queryAll="queryAll" :queryPublic="queryPublic" />
      </a-col>
    </a-row>
  </div>
</template>

<script setup lang="ts">
import SpaceUsageAnalyze from '@/components/analyze/SpaceUsageAnalyze.vue'
import SpaceCategoryAnalyze from '@/components/analyze/SpaceCategoryAnalyze.vue'
import SpaceTagAnalyze from '@/components/analyze/SpaceTagAnalyze.vue'
import SpaceSizeAnalyze from '@/components/analyze/SpaceSizeAnalyze.vue'
import SpaceUserAnalyze from '@/components/analyze/SpaceUserAnalyze.vue'
import SpaceRankAnalyze from '@/components/analyze/SpaceRankAnalyze.vue'
import { useRoute } from 'vue-router'
import { computed } from 'vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'

const route = useRoute()
const spaceId = computed(() => route.query?.spaceId as string)
const queryAll = computed(() => !!route.query?.queryAll)
const queryPublic = computed(() => !!route.query?.queryPublic)
const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser
const isAdmin = computed(() => loginUser.userRole === 'admin')
</script>

<style scoped>
#spaceAnalyzePage {
  display: grid;
  gap: 18px;
}

.analyze-hero {
  padding: 24px;
}
</style>
