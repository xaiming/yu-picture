<template>
  <div id="userLoginPage" class="auth-page">
    <div class="auth-card anime-panel">
      <p class="eyebrow">Welcome Back</p>
      <h2>登录星璃云图</h2>
      <p class="desc">继续管理你的二次元图片灵感库</p>
      <a-form :model="formState" name="basic" autocomplete="off" @finish="handleSubmit">
        <a-form-item name="userAccount" :rules="[{ required: true, message: '请输入账号' }]">
          <a-input v-model:value="formState.userAccount" placeholder="请输入账号" />
        </a-form-item>
        <a-form-item
          name="userPassword"
          :rules="[
            { required: true, message: '请输入密码' },
            { min: 8, message: '密码长度不能小于 8 位' },
          ]"
        >
          <a-input-password v-model:value="formState.userPassword" placeholder="请输入密码" />
        </a-form-item>
        <div class="tips">
          没有账号？
          <RouterLink to="/user/register">去注册</RouterLink>
        </div>
        <a-form-item>
          <a-button type="primary" html-type="submit" block>登录</a-button>
        </a-form-item>
      </a-form>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive } from 'vue'
import { userLoginUsingPost } from '@/api/userController.ts'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { message } from 'ant-design-vue'
import router from '@/router'

const formState = reactive<API.UserLoginRequest>({
  userAccount: '',
  userPassword: '',
})

const loginUserStore = useLoginUserStore()

const handleSubmit = async (values: API.UserLoginRequest) => {
  const res = await userLoginUsingPost(values)
  if (res.data.code === 0 && res.data.data) {
    await loginUserStore.fetchLoginUser()
    message.success('登录成功')
    router.push({
      path: '/gallery',
      replace: true,
    })
  } else {
    message.error('登录失败：' + res.data.message)
  }
}
</script>

<style scoped>
.auth-page {
  display: grid;
  min-height: calc(100vh - 210px);
  place-items: center;
}

.auth-card {
  width: min(100%, 420px);
  padding: 32px;
  background:
    radial-gradient(circle at 90% 0%, rgba(255, 157, 203, 0.3), transparent 36%),
    rgba(255, 255, 255, 0.76);
}

.eyebrow {
  margin: 0 0 8px;
  color: var(--xl-primary);
  font-weight: 800;
}

h2 {
  margin: 0;
  color: var(--xl-text);
  font-size: 30px;
}

.desc {
  margin: 10px 0 24px;
  color: var(--xl-muted);
}

.tips {
  margin-bottom: 16px;
  color: var(--xl-muted);
  text-align: right;
}
</style>
