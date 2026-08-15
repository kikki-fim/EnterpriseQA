<template>
  <div class="login-page">
    <!-- 左侧 40%：灰色品牌区 -->
    <div class="brand-area">
      <div class="brand-badge">
        <div class="brand-logo">
          <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor">
            <path d="M4 3h7v7H4zM13 3h7v7h-7zM4 12h7v7H4zM13 12h7v7h-7z"/>
          </svg>
        </div>
        <span class="brand-en">ENTERPRISE KB</span>
      </div>
      <h1 class="brand-title">企业知识库</h1>
      <h3 class="brand-title brand-subtitle">智能问答系统</h3>
      <p class="brand-desc">基于 RAG 技术的企业级知识管理 与智能检索平台</p>

      <!-- 功能特性列表 -->
      <ul class="feature-list">
        <li>多知识库管理</li>
        <li>文档智能向量化</li>
        <li>AI 精准问答</li>
      </ul>
    </div>

    <!-- 右侧 60%：白色区域 + 垂直居中登录卡片 -->
    <div class="form-area">
      <div class="login-card">
        <h2 class="form-title">欢迎回来</h2>
        <p class="form-sub">请登录您的账号以继续使用系统</p>

        <el-form
          ref="formRef"
          :model="loginForm"
          :rules="rules"
          @keyup.enter="handleLogin"
        >
          <el-form-item prop="username" class="field-item">
            <label class="field-label">账号</label>
            <el-input
              v-model="loginForm.username"
              placeholder="请输入账号"
              size="default"
            />
          </el-form-item>
          <el-form-item prop="password" class="field-item">
            <label class="field-label">密码</label>
            <el-input
              v-model="loginForm.password"
              type="password"
              placeholder="请输入密码"
              size="default"
              show-password
            />
          </el-form-item>
        </el-form>

        <button class="login-btn" :disabled="loading" @click="handleLogin">
          {{ loading ? '登录中...' : '登 录' }}
        </button>

        <p class="test-hint">测试账号：admin  123456（管理员）｜ user1  123456（用户）</p>
      </div>
    </div>
  </div>
</template>

<script setup>
/**
 * 登录页面 - Notion 灰白黑极简风格
 * 左右 4:6 分栏布局，左侧品牌区含功能列表
 * 支持用户名密码登录，登录成功后跳转至对应首页
 */
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { login } from '../api/auth'
import { useUserStore } from '../stores/user'

const router = useRouter()
const userStore = useUserStore()
const formRef = ref(null)
const loading = ref(false)

/** 登录表单 */
const loginForm = reactive({
  username: '',
  password: ''
})

/** 表单校验规则 */
const rules = {
  username: [{ required: true, message: '请输入账号', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
}

/** 处理登录 */
async function handleLogin() {
  const valid = await formRef.value.validate().catch(() => false)
  if (!valid) return

  loading.value = true
  try {
    const res = await login(loginForm)
    userStore.setLoginInfo(res.data)
    ElMessage.success('登录成功')
    // 管理员跳转数据概览，普通用户跳转智能问答
    if (res.data.user.role === 'admin') {
      router.push('/home')
    } else {
      router.push('/chat')
    }
  } catch (err) {
    // 错误已在拦截器中处理
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* ===== 页面根容器：左右分栏 4:6 ===== */
.login-page {
  min-height: 100vh;
  display: flex;
  flex-direction: row;
  background: #ffffff;
  font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", "PingFang SC",
    "Microsoft YaHei", sans-serif;
  color: #37352f;
}

/* ===== 左侧 40%：灰色品牌区 ===== */
.brand-area {
  width:35%;
  background: #f0efed;
  padding: 120px 56px;        /* 只调这个值，控制内容离顶部多远 */
  display: flex;
  flex-direction: column;
  /* justify-content: center;  改为下面这行 */
  justify-content: flex-start;  /* 内容从顶部开始排 */
  gap:22px;
}


.brand-badge {
  display: inline-flex;
  align-items: center;
  gap: 12px;
}

.brand-logo {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  background: #37352f;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ffffff;
}

.brand-en {
  font-size: 17px;
  font-weight: 600;
  color: #9b9a97;
  letter-spacing: 1px;
}

.brand-title {
  font-size: 45px;
  font-weight: 550;
  color: #37352f;
  margin: 0;
  line-height: 1.25;
}

.brand-subtitle {
  margin-top: -14px;
  font-size: 42px;
  font-weight: 350;
}

.brand-desc {
  font-size: 19px;
  font-weight: 400;
  color: #787774;
  margin: 0;
  line-height: 1.65;
  max-width: 300px;
}

/* ===== 功能特性列表 ===== */
.feature-list {
  list-style: none;
  padding: 0;
  margin: 30px 0 0;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.feature-list li {
  font-size: 18px;
  font-weight: 500;
  color: #807f7c;
  padding-left: 20px;
  position: relative;
  line-height: 1.5;
}

.feature-list li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 11px;
  width: 8px;
  height: 8px;
  border-radius: 20%;
  background: #37352f;
}

/* ===== 右侧 60%：白色区域 + 垂直居中卡片 ===== */
.form-area {
  width: 65%;
  background: #ffffffc8;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 48px 36px;
}

.login-card {
  width: 500px;
  max-width: 120%;
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.08);
  border-radius: 6px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
  padding: 42px 38px 34px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 18px;
}

.form-title {
  font-size: 28px;
  font-weight: 700;
  color: #37352f;
  margin: 0;
  align-self: flex-start;
}

.form-sub {
  font-size: 15px;
  font-weight: 400;
  color: #9b9a97;
  margin: -12px 0 0;
  align-self: flex-start;
}

/* ===== 表单项 & 输入框 ===== */
.field-item {
  margin-bottom: 8px;
  width: 100%;
}

.field-label {
  font-size: 15px;
  font-weight: 600;
  color: #787774;
  margin-bottom: 6px;
  display: block;
}

.field-item :deep(.el-input__wrapper) {
  height: 46px;
  border-radius: 6px;
  border: 1px solid rgba(0, 0, 0, 0.12);
  box-shadow: none !important;
  background: #ffffff;
  transition: border-color 0.2s;
}

.field-item :deep(.el-input__wrapper:hover) {
  border-color: rgba(55, 53, 47, 0.25);
}

.field-item :deep(.el-input__wrapper.is-focus) {
  border-color: #37352f;
  box-shadow: none !important;
}

.field-item :deep(.el-input__inner) {
  font-size: 16px;
  font-weight: 400;
  color: #37352f;
}

.field-item :deep(.el-input__inner::placeholder) {
  color: #bfbec4;
}

.field-item :deep(.el-form-item__error) {
  font-size: 12px;
  padding-top: 2px;
}

/* 登录按钮 */
.login-btn {
  height: 44px;
  padding: 0 38px;
  border: 0;
  border-radius: 6px;
  background: #37352f;
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.login-btn:hover {
  background: #4a4741;
}

.login-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.test-hint {
  font-size: 13px;
  font-weight: 400;
  color: #9b9a97;
  white-space: nowrap;
  margin: 0;
  line-height: 1.5;
}
</style>
