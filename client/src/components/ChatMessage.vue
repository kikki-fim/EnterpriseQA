<template>
  <!-- 对话消息气泡组件 -->
  <div class="message-wrapper" :class="{ 'is-user': isUser }">
    <div class="avatar">
      <el-avatar :size="36" :icon="isUser ? UserFilled : Monitor" :style="avatarStyle" />
    </div>
    <div class="bubble" :class="{ 'user-bubble': isUser, 'ai-bubble': !isUser }">
      <div class="message-text">{{ message.content }}</div>
      <!-- AI回答时显示参考来源 -->
      <div v-if="!isUser && message.sources?.length" class="sources">
        <div class="sources-title">参考来源：</div>
        <el-tag
          v-for="(src, i) in message.sources"
          :key="i"
          size="small"
          type="info"
          class="source-tag"
        >
          {{ src.file_name }}
        </el-tag>
      </div>
    </div>
  </div>
</template>

<script setup>
/**
 * 对话消息气泡组件
 * 区分用户消息和AI回答，AI回答可展示参考来源
 */
import { computed } from 'vue'
import { UserFilled, Monitor } from '@element-plus/icons-vue'

const props = defineProps({
  /** 消息对象 { role: 'user'|'ai', content: string, sources?: array } */
  message: { type: Object, required: true }
})

/** 是否为用户消息 */
const isUser = computed(() => props.message.role === 'user')

/** 头像样式 */
const avatarStyle = computed(() => {
  if (isUser.value) {
    // 用户：白底黑线
    return {
      backgroundColor: '#ffffff',
      color: '#1A1A1A',
      border: '1px solid #d0d0cf'
    }
  }
  // AI：黑底白线
  return {
    backgroundColor: '#1A1A1A',
    color: '#ffffff'
  }
})
</script>

<style scoped>
.message-wrapper {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
  align-items: flex-start;
}

.message-wrapper.is-user {
  flex-direction: row-reverse;
}

.bubble {
  max-width: 70%;
  padding: 12px 16px;
  border-radius: 12px;
  line-height: 1.6;
  word-break: break-word;
  white-space: pre-wrap;
}

.user-bubble {
  background: #e8f1fd;
  color: #1A1A1A;
  border: 1px solid #bdd6f0;
  border-top-right-radius: 4px;
  box-shadow: 0 2px 8px rgba(65, 105, 168, 0.12);
}

.ai-bubble {
  background: #ffffff;
  color: #1A1A1A;
  border: 1px solid #1A1A1A;
  border-top-left-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}



.ai-bubble {
  background: #ffffff;
  color: #1e1f1f;
  border: 1px solid #e3dfdf;
  border-top-left-radius: 4px;
}


.message-text {
  font-size: 14px;
}

/* 用户头像图标颜色 */
.avatar :deep(.el-avatar .el-icon) {
  color: inherit;
}

.sources {
  margin-top: 10px;
  padding-top: 8px;
  border-top: 1px solid #e4e7ed;
}

.sources-title {
  font-size: 12px;
  color: #909399;
  margin-bottom: 4px;
}

.source-tag {
  margin-right: 4px;
  margin-bottom: 4px;
  color: #0b8f7a;
  background: #e0f5f1;
  border-color: #b5e8de;
}

</style>
