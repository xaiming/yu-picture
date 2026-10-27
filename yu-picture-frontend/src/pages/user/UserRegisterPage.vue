<template>
  <div id="userRegisterPage" class="auth-page">
    <div class="auth-card anime-panel">
      <p class="eyebrow">Create Account</p>
      <h2>加入星璃云图</h2>
      <p class="desc">创建你的二次元图片收藏空间</p>
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
        <a-form-item
          name="checkPassword"
          :rules="[
            { required: true, message: '请输入确认密码' },
            { min: 8, message: '确认密码长度不能小于 8 位' },
          ]"
        >
          <a-input-password v-model:value="formState.checkPassword" placeholder="请再次输入密码" />
        </a-form-item>
        <div class="tips">
          已有账号？
          <RouterLink to="/user/login">去登录</RouterLink>
        </div>
        <a-form-item>
          <a-button type="primary" html-type="submit" block>注册</a-button>
        </a-form-item>
      </a-form>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive } from 'vue'
import { userRegisterUsingPost } from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import router from '@/router'

const formState = reactive<API.UserRegisterRequest>({
  userAccount: '',
  userPassword: '',
  checkPassword: '',
})

const handleSubmit = async (values: API.UserRegisterRequest) => {
  if (values.userPassword !== values.checkPassword) {
    message.error('两次输入的密码不一致')
    return
  }
  const res = await userRegisterUsingPost(values)
  if (res.data.code === 0 && res.data.data) {
    message.success('注册成功')
    router.push({
      path: '/user/login',
      replace: true,
    })
  } else {
    message.error('注册失败：' + res.data.message)
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
    radial-gradient(circle at 90% 0%, rgba(104, 200, 255, 0.3), transparent 36%),
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
