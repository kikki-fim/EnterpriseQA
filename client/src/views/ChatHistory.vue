<template>
  <!-- 对话历史页面 -->
  <div class="page-container">

    <!-- 筛选栏 -->
    <el-card shadow="never" class="search-bar">
      <el-row :gutter="16" align="middle">
        <el-col :span="8">
          <el-select
            v-model="queryParams.kb_id"
            placeholder="按知识库筛选"
            clearable
            @change="loadList"
            style="width: 100%"
          >
            <el-option
              v-for="kb in kbOptions"
              :key="kb.id"
              :label="kb.kb_name"
              :value="kb.id"
            />
          </el-select>
        </el-col>
      </el-row>
    </el-card>

    <!-- 历史记录表格 -->
    <el-card shadow="never" class="table-card">
      <el-table :data="tableData" v-loading="loading" stripe>
        <el-table-column prop="id" label="ID" width="60" />
        <el-table-column prop="question" label="问题" min-width="250" show-overflow-tooltip />
        <el-table-column prop="answer" label="回答" min-width="300" show-overflow-tooltip />
        <el-table-column prop="kb_name" label="知识库" width="130" />
        <el-table-column prop="username" label="提问者" width="100" />
        <el-table-column prop="create_time" label="时间" width="170" />
        <el-table-column label="操作" width="80" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" link @click="showDetail(row)">详情</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          v-model:current-page="queryParams.page"
          v-model:page-size="queryParams.page_size"
          :total="total"
          :page-sizes="[10, 20, 50]"
          layout="total, sizes, prev, slot, next"
          @change="loadList"
        >
          <span class="page-counter">{{ queryParams.page }}/{{ totalPages }}</span>
        </el-pagination>
      </div>
    </el-card>

    <!-- 详情对话框 -->
    <el-dialog
      v-model="detailVisible"
      class="kb-dialog"
      title="对话详情"
      width="650px"
    >
      <div class="detail-content" v-if="currentChat">
        <div class="detail-item">
          <div class="detail-label">提问：</div>
          <div class="detail-value question">{{ currentChat.question }}</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">回答：</div>
          <div class="detail-value answer">{{ currentChat.answer }}</div>
        </div>
        <div class="detail-item" v-if="currentChat.source_docs?.length">
          <div class="detail-label">参考来源：</div>
          <div class="detail-value">
            <el-tag
              v-for="(src, i) in currentChat.source_docs"
              :key="i"
              size="small"
              class="source-tag"
            >
              {{ src.file_name }}
            </el-tag>
          </div>
        </div>
        <div class="detail-item">
          <div class="detail-label">知识库：</div>
          <div class="detail-value">{{ currentChat.kb_name }}</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">时间：</div>
          <div class="detail-value">{{ currentChat.create_time }}</div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
/**
 * 对话历史页面
 * 展示用户的历史问答记录，支持按知识库筛选和查看详情
 */
import { ref, reactive, computed, onMounted } from 'vue'
import { getChatHistory } from '../api/chat'
import { getAllKB } from '../api/knowledge'

const loading = ref(false)
const detailVisible = ref(false)
const tableData = ref([])
const total = ref(0)
const kbOptions = ref([])
const currentChat = ref(null)

const queryParams = reactive({ page: 1, page_size: 10, kb_id: null })

/** 总页数 */
const totalPages = computed(() => {
  return Math.max(1, Math.ceil(total.value / queryParams.page_size))
})

async function loadKBOptions() {
  const res = await getAllKB()
  kbOptions.value = res.data
}

async function loadList() {
  loading.value = true
  try {
    const res = await getChatHistory(queryParams)
    tableData.value = res.data.list
    total.value = res.data.total
  } finally {
    loading.value = false
  }
}

function showDetail(row) {
  currentChat.value = row
  detailVisible.value = true
}

onMounted(() => {
  loadKBOptions()
  loadList()
})
</script>

<style scoped>
.page-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'SF Pro Text',
    'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-weight: 400;
  color: #1A1A1A;
}

.search-bar {
  border-radius: 8px;
}

.table-card {
  border-radius: 8px;
}

.pagination {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 12px;
  margin-top: 16px;
}

.page-counter {
  font-size: 13px;
  color: #000000;
  font-weight: 300;
  padding: 0 8px;
}

/* 文字层级规范 */
.page-container :deep(.el-card__body) {
  color: #1A1A1A;
  font-weight: 400;
}

.table-card :deep(.el-table th) {
  color: #37352F;
  font-weight: 400;
  font-size: 16px;
}

.table-card :deep(.el-table td) {
  color: #747474;
  font-weight: 400;
}

.table-card :deep(.el-table .cell) {
  font-weight: 400;
}

/* 筛选下拉框 */
.search-bar :deep(.el-select .el-input__wrapper) {
  box-shadow: 0 0 0 1px #d0d0cf inset;
  border-radius: 6px;
}
.search-bar :deep(.el-select .el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 1px #37352F inset;
}
.search-bar :deep(.el-select .el-input__inner) {
  color: #1A1A1A;
  font-weight: 400;
}

/* 详情按钮颜色（与删除按钮一致） */
.table-card :deep(.el-button--primary.is-link) {
  color: #ff8585;
}
.table-card :deep(.el-button--primary.is-link:hover) {
  color: #b53c3c;
}

.pagination :deep(.el-pagination) {
  color: #1A1A1A;
  font-weight: 400;
  font-size: 12px;
}

/* 详情内容 */
.detail-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.detail-item {
  display: flex;
  gap: 8px;
}

.detail-label {
  font-weight: 400;
  color: #1A1A1A;
  white-space: nowrap;
  min-width: 70px;
}

.detail-value {
  color: #747474;
  line-height: 1.6;
  word-break: break-all;
}

.detail-value.question {
  color: #37352F;
  font-weight: 500;
}

.detail-value.answer {
  background: #f0efed;
  padding: 12px;
  border-radius: 6px;
  white-space: pre-wrap;
  color: #1A1A1A;
}

.source-tag {
  margin-right: 6px;
  margin-bottom: 4px;
  color: #9b8afa;
  background: rgba(155, 138, 250, 0.12);
  border-color: rgba(155, 138, 250, 0.35);
}

</style>

<style>
/* 全局黑白灰 - 对话框、下拉框等 teleport 组件 */
:root {
  --el-color-primary: #37352F;
  --el-color-primary-light-3: #6b6964;
  --el-color-primary-light-5: #9B9A97;
  --el-color-primary-light-7: #c9c8c5;
  --el-color-primary-light-8: #d0d0cf;
  --el-color-primary-light-9: #e8e8e6;
  --el-color-primary-dark-2: #2c2a26;
}

/* 对话框 */
.kb-dialog {
  border-radius: 8px;
}
.kb-dialog .el-dialog__title {
  color: #37352F;
  font-weight: 400;
}
.kb-dialog .el-dialog__headerbtn .el-dialog__close {
  color: #9B9A97;
}
.kb-dialog .el-dialog__headerbtn:hover .el-dialog__close {
  color: #37352F;
}
.kb-dialog .el-button {
  font-weight: 400;
}

/* 下拉框选项 */
.el-select-dropdown__item.selected {
  color: #37352F;
  font-weight: 500;
}
.el-select-dropdown__item.hover {
  background-color: #f0efed;
}

/* 分页页码不加粗 */
.el-pagination .el-pager li.is-active {
  font-weight: 400;
}

/* 表格省略提示 tooltip 背景色 + 文字颜色 */
.el-popper.is-dark {
  background: #e4e4e4;
  border-color: #e4e4e4;
  color: #000000;   /* 文字颜色，改成你想要的 */
}
.el-popper.is-dark .el-popper__arrow::before {
  background: #e4e4e4;
}


</style>
