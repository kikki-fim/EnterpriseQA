<template>
  <!-- 后台主布局：左侧菜单 + 顶栏 + 内容区 -->
  <el-container class="layout-container">
    <!-- 左侧菜单栏 -->
    <el-aside
      :width="isCollapse ? '68px' : '220px'"
      class="aside"
      :class="{ collapsed: isCollapse }"
    >
      <div class="logo">
        <el-icon :size="22"><ChatDotSquare /></el-icon>
        <span v-show="!isCollapse" class="logo-text">企业知识库</span>
      </div>

      <nav class="menu-list">
        <template v-if="userStore.isAdmin">
          <div
            v-for="item in adminMenus"
            :key="item.path"
            class="menu-item"
            :class="{ active: route.path === item.path }"
            @click="router.push(item.path)"
          >
            <span class="menu-icon" v-html="item.icon"></span>
            <span v-show="!isCollapse" class="menu-label">{{ item.label }}</span>
          </div>
          <div v-if="!isCollapse" class="menu-divider"></div>
        </template>

        <div
          v-for="item in commonMenus"
          :key="item.path"
          class="menu-item"
          :class="{ active: route.path === item.path }"
          @click="router.push(item.path)"
        >
          <span class="menu-icon" v-html="item.icon"></span>
          <span v-show="!isCollapse" class="menu-label">{{ item.label }}</span>
        </div>
      </nav>
    </el-aside>

    <!-- 右侧内容区 -->
    <el-container>
      <el-header class="header">
        <div class="header-left">
          <el-icon class="collapse-btn" @click="isCollapse = !isCollapse">
            <Fold v-if="!isCollapse" />
            <Expand v-else />
          </el-icon>
          <span class="page-title">{{ $route.meta.title }}</span>
        </div>
        <div class="header-right">
          <el-dropdown @command="handleCommand">
            <span class="user-info">
              <el-avatar :size="32" :icon="UserFilled" />
              <span class="username">{{ userStore.userInfo?.nickname }}</span>
            </span>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item command="logout">
                  <el-icon><SwitchButton /></el-icon>退出登录
                </el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </el-header>

      <el-main class="main">
        <router-view />
      </el-main>
    </el-container>
  </el-container>
</template>

<script setup>
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '../stores/user'
import {
  ChatDotSquare, Fold, Expand, UserFilled, SwitchButton
} from '@element-plus/icons-vue'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()

const isCollapse = ref(false)

/** 四宫格方块 */
const gridIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><rect x="3" y="3" width="7.6" height="7.6" rx="1.6"/><rect x="13.4" y="3" width="7.6" height="7.6" rx="1.6"/><rect x="3" y="13.4" width="7.6" height="7.6" rx="1.6"/><rect x="13.4" y="13.4" width="7.6" height="7.6" rx="1.6"/></svg>`
/** 书本 */
const bookIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M6 2h9a3 3 0 0 1 3 3v14a3 3 0 0 1-3 3H6a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2z"/><path d="M14 2v17" stroke="#f7f6f3" stroke-width="1.6"/></svg>`
/** 文件 */
const fileIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M7 2h6l5 5v13a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2z"/><path d="M10 12h6M10 16h6" stroke="#f7f6f3" stroke-width="1.6" stroke-linecap="round"/></svg>`
/** 双人 */
const usersIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><circle cx="9" cy="8" r="3.4"/><path d="M3.2 19.5c.5-3.5 2.7-5.5 5.8-5.5s5.3 2 5.8 5.5z"/><circle cx="16.6" cy="9.2" r="2.6"/><path d="M15.4 14.4c2.4.3 4.2 1.8 4.8 4.6H14c.4-2.6 1.9-4.2 4.3-4.6z" opacity=".85"/></svg>`
/** 对话气泡 */
const chatIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M4 4.5h16a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2h-7l-4.6 3.7V16.5H4a2 2 0 0 1-2-2v-8a2 2 0 0 1 2-2z"/></svg>`
/** 时钟 */
const clockIcon = `<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><circle cx="12" cy="12" r="9"/><path d="M12 6.5V12l3.6 2.2" stroke="#f7f6f3" stroke-width="2" stroke-linecap="round" fill="none"/></svg>`

const adminMenus = [
  { label: '数据概览', icon: gridIcon, path: '/home' },
  { label: '知识库管理', icon: bookIcon, path: '/knowledge-base' },
  { label: '文档管理', icon: fileIcon, path: '/document' },
  { label: '用户管理', icon: usersIcon, path: '/user-manage' }
]

const commonMenus = [
  { label: '智能问答', icon: chatIcon, path: '/chat' },
  { label: '对话历史', icon: clockIcon, path: '/chat-history' }
]

function handleCommand(command) {
  if (command === 'logout') {
    userStore.logout()
    router.push('/login')
  }
}
</script>

<style scoped>
.layout-container {
  height: 100vh;
}

.aside {
  background-color: #ffffff;
  border-right: 1px solid #e9e9e7;
  transition: width 0.25s ease;
  overflow: hidden;
}

.logo {
  height: 61px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 18px;
  color: #545351;
  border-bottom: 2px solid #e9e9e7;
}

.logo-text {
  font-size: 19px;
  font-weight: 540;
  white-space: nowrap;
}

.menu-list {
  padding: 12px 10px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.menu-item {
  display: flex;
  align-items: center;
  gap: 8px;
  height: 44px;
  padding: 0 12px;
  border-radius: 4px;
  font-size: 16px;
  color: #37352f;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.15s;
  user-select: none;
}

.menu-item:hover {
  background: #f0efed;
}

.menu-item.active {
  background: #e8e8e6;
}

.menu-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  color: #9b9a97;
  transition: color 0.15s;
}

.menu-item:hover .menu-icon,
.menu-item.active .menu-icon {
  color: #37352f;
}

.menu-label {
  white-space: nowrap;
}

.menu-divider {
  height: 1px;
  background: #e9e9e7;
  margin: 8px 4px;
}

.aside.collapsed .menu-item {
  justify-content: center;
  padding: 0;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #e9e9e7;
  background: #fff;
  padding: 0 20px;
  height: 60px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.collapse-btn {
  font-size: 20px;
  cursor: pointer;
  color: #606266;
}

.page-title {
  font-size: 16px;
  font-weight: 500;
  color: #303133;
}

.header-right .user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.username {
  font-size: 14px;
  color: #606266;
}

.main {
  background: #efefef;
  padding: 20px;
  overflow-y: auto;
}
</style>
