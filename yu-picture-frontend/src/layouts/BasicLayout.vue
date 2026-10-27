<template>
  <div id="basicLayout">
    <a-layout class="app-shell">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout class="body-shell">
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>
      <a-layout-footer class="footer">
        <RouterLink class="watch-mode-link" to="/">观赏模式</RouterLink>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import { RouterLink } from 'vue-router'
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'
</script>

<style scoped>
#basicLayout {
  min-height: 100vh;
  background:
    radial-gradient(circle at 8% 12%, rgba(255, 157, 203, 0.24), transparent 25%),
    radial-gradient(circle at 92% 0%, rgba(104, 200, 255, 0.28), transparent 28%),
    linear-gradient(135deg, #f7fbff 0%, #f1edff 52%, #fff7fc 100%);
}

.app-shell {
  min-height: 100vh;
  background: transparent;
}

.header {
  position: sticky;
  top: 0;
  z-index: 10;
  height: 68px;
  padding: 10px 22px;
  border-bottom: 1px solid rgba(124, 108, 255, 0.14);
  background: rgba(255, 255, 255, 0.68);
  backdrop-filter: blur(18px);
}

.body-shell {
  min-height: calc(100vh - 116px);
  background: transparent;
}

.sider {
  padding: 18px 0 72px 18px;
}

.content {
  min-width: 0;
  margin: 18px 18px 72px;
  padding: 26px;
  border: 1px solid rgba(124, 108, 255, 0.16);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.56);
  box-shadow: 0 28px 70px rgba(112, 110, 255, 0.13);
  backdrop-filter: blur(18px);
}

.footer {
  position: fixed;
  right: 22px;
  bottom: 18px;
  z-index: 20;
  width: auto;
  padding: 0;
  background: transparent;
}

.watch-mode-link {
  display: inline-flex;
  min-width: 118px;
  height: 40px;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(124, 108, 255, 0.28);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.72);
  box-shadow: 0 16px 34px rgba(124, 108, 255, 0.2);
  color: #6f5cff;
  font-weight: 700;
  backdrop-filter: blur(14px);
}

@media (max-width: 768px) {
  .header {
    height: auto;
    padding: 10px 14px;
  }

  .sider {
    padding: 0;
  }

  .content {
    margin: 12px 12px 72px;
    padding: 16px;
  }
}
</style>
