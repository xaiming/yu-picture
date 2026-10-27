<template>
  <div id="app">
    <a-config-provider :locale="zhCN">
      <router-view v-if="route.meta.fullscreen" />
      <BasicLayout v-else />
    </a-config-provider>
  </div>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router'
import zhCN from 'ant-design-vue/es/locale/zh_CN';
import dayjs from 'dayjs';
import 'dayjs/locale/zh-cn';
dayjs.locale('zh-cn');

import BasicLayout from '@/layouts/BasicLayout.vue'
const route = useRoute()
// import { healthUsingGet } from '@/api/mainController.ts'
// import {useLoginUserStore} from "@/stores/useLoginUserStore.ts";

// 已经改为在权限校验文件中获取
// const loginUserStore = useLoginUserStore()
// loginUserStore.fetchLoginUser()

// healthUsingGet().then((res) => {
//   console.log(res)
// })
</script>

<style scoped></style>
