<template>
  <!-- 知识库管理页面 -->
  <div class="page-container">

    <!-- 操作栏 -->
    <el-card shadow="never" class="search-bar">
      <el-row :gutter="16" align="middle">
        <el-col :span="8">
          <div class="search-wrapper">
            <el-input
              v-model="queryParams.keyword"
              placeholder="搜索知识库名称"
              clearable
              :prefix-icon="Search"
              @clear="loadList"
              @keyup.enter="loadList"
            />
          </div>
            
        </el-col>
        <el-col :span="16" style="text-align: right">
          <div class="btn-wrapper">
            <el-button class="add-btn" :icon="Plus" @click="handleAdd">新增知识库</el-button>
            <div class="btn-en">Add Knowledge Base</div>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <!-- 数据表格 -->
    <el-card shadow="never" class="table-card">
      <el-table :data="tableData" v-loading="loading" stripe>
        <el-table-column prop="id" label="ID" width="60" />
        <el-table-column prop="kb_name" label="知识库名称" min-width="150" />
        <el-table-column prop="description" label="描述" min-width="200" show-overflow-tooltip />
        <el-table-column prop="doc_count" label="文档数" width="80" align="center" />
        <el-table-column prop="creator_name" label="创建者" width="100" />
        <el-table-column prop="create_time" label="创建时间" width="170" />
        <el-table-column label="操作" width="160" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" link @click="handleEdit(row)">编辑</el-button>
            <el-popconfirm title="确认删除该知识库？" @confirm="handleDelete(row.id)">
              <template #reference>
                <el-button type="danger" link>删除</el-button>
              </template>
            </el-popconfirm>
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

    <!-- 新增/编辑对话框 -->
    <el-dialog
      v-model="dialogVisible"
      class="kb-dialog"
      :title="isEdit ? '编辑知识库' : '新增知识库'"
      width="500px"
    >
      <el-form ref="formRef" :model="formData" :rules="rules" label-width="100px">
        <el-form-item label="知识库名称" prop="kb_name">
          <el-input v-model="formData.kb_name" placeholder="请输入知识库名称" />
        </el-form-item>
        <el-form-item label="描述" prop="description">
          <el-input
            v-model="formData.description"
            type="textarea"
            :rows="3"
            placeholder="请输入知识库描述"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button class="cancel-btn" @click="dialogVisible = false">取消</el-button>
        <el-button class="confirm-btn" :loading="submitLoading" @click="handleSubmit">确定</el-button>
      </template>

    </el-dialog>
  </div>
</template>

<script setup>
/**
 * 知识库管理页面
 * 支持知识库的列表查询、新增、编辑和删除操作
 */
import { ref, reactive, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { Search, Plus } from '@element-plus/icons-vue'
import { getKBList, createKB, updateKB, deleteKB } from '../api/knowledge'

const loading = ref(false)
const submitLoading = ref(false)
const dialogVisible = ref(false)
const isEdit = ref(false)
const tableData = ref([])
const total = ref(0)
const formRef = ref(null)

/** 查询参数 */
const queryParams = reactive({ page: 1, page_size: 10, keyword: '' })

/** 总页数 */
const totalPages = computed(() => {
  return Math.max(1, Math.ceil(total.value / queryParams.page_size))
})

/** 表单数据 */
const formData = reactive({ id: null, kb_name: '', description: '' })

/** 文档总数（从列表汇总） */
const totalDocCount = computed(() => {
  return tableData.value.reduce((sum, row) => sum + (row.doc_count || 0), 0)
})

/** 表单校验 */
const rules = {
  kb_name: [{ required: true, message: '请输入知识库名称', trigger: 'blur' }]
}

/** 加载知识库列表 */
async function loadList() {
  loading.value = true
  try {
    const res = await getKBList(queryParams)
    tableData.value = res.data.list
    total.value = res.data.total
  } finally {
    loading.value = false
  }
}

/** 打开新增对话框 */
function handleAdd() {
  isEdit.value = false
  Object.assign(formData, { id: null, kb_name: '', description: '' })
  dialogVisible.value = true
}

/** 打开编辑对话框 */
function handleEdit(row) {
  isEdit.value = true
  Object.assign(formData, { id: row.id, kb_name: row.kb_name, description: row.description })
  dialogVisible.value = true
}

/** 提交表单 */
async function handleSubmit() {
  const valid = await formRef.value.validate().catch(() => false)
  if (!valid) return

  submitLoading.value = true
  try {
    if (isEdit.value) {
      await updateKB(formData.id, formData)
      ElMessage.success('更新成功')
    } else {
      await createKB(formData)
      ElMessage.success('创建成功')
    }
    dialogVisible.value = false
    loadList()
  } finally {
    submitLoading.value = false
  }
}

/** 删除知识库 */
async function handleDelete(id) {
  await deleteKB(id)
  ElMessage.success('删除成功')
  loadList()
}

onMounted(() => loadList())
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
  color: #0d0d0d;
  font-weight: 300;
  padding: 0 8px;
}


/* 搜索框下方英文 */
.search-wrapper {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}
.search-en {
  font-size: 11px;
  color: #9CA3AF;
  font-weight: 300;
  line-height: 1;
}

/* 按钮下方英文 */
.btn-wrapper {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.btn-en {
  font-size: 11px;
  color: #9CA3AF;
  font-weight: 300;
  line-height: 1;
}

.add-btn {
  --el-button-bg-color: #0c0c0c;
  --el-button-border-color: #c7c7c7;
  --el-button-text-color: #f9f3f3;
  --el-button-hover-bg-color: #c3c2c2;
  --el-button-hover-border-color: #a6a6a6;
  --el-button-hover-text-color: #201f1f;
  --el-button-active-bg-color: #d4d4d4;
  --el-button-active-border-color: #c9c9c9;
  --el-button-active-text-color: #151515;
  border-radius: 4px;
  font-weight: 400;
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

.search-bar :deep(.el-input__inner) {
  color: #1A1A1A;
  font-weight: 400;
}
.search-bar :deep(.el-input__inner::placeholder) {
  color: #9CA3AF;
  font-weight: 300;
}

.pagination :deep(.el-pagination) {
  color: #1A1A1A;
  font-weight: 400;
  font-size: 12px;
}

/* 编辑按钮颜色 */
.table-card :deep(.el-button--primary.is-link) {
  color: #9b8afa;
}
.table-card :deep(.el-button--primary.is-link:hover) {
  color: #684ccd;
}

/* 删除按钮颜色 */
.table-card :deep(.el-button--danger.is-link) {
  color: #ff8585;
}
.table-card :deep(.el-button--danger.is-link:hover) {
  color: #b53c3c;
}

</style>

<style>
/* 全局黑白灰 - 对话框、下拉框、弹出确认等 teleport 组件 */
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
.kb-dialog .el-form-item__label {
  color: #1A1A1A;
  font-weight: 400;
}
.kb-dialog .el-input__wrapper {
  background: #fff;
  box-shadow: 0 0 0 1px #d0d0cf inset;
  border-radius: 6px;
}
.kb-dialog .el-input__wrapper:hover {
  box-shadow: 0 0 0 1px #9B9A97 inset;
}
.kb-dialog .el-input__wrapper.is-focus {
  box-shadow: 0 0 0 1px #37352F inset;
}
.kb-dialog .el-input__inner {
  color: #1A1A1A;
  font-weight: 400;
}
.kb-dialog .el-input__inner::placeholder {
  color: #9CA3AF;
  font-weight: 300;
}
.kb-dialog .el-textarea__inner {
  color: #1A1A1A;
  font-weight: 400;
  background: #fff;
  border: 1px solid #d0d0cf;
  border-radius: 6px;
  box-shadow: none;
}
.kb-dialog .el-textarea__inner:hover {
  border-color: #9B9A97;
}
.kb-dialog .el-textarea__inner:focus {
  border-color: #37352F;
  box-shadow: none;
}
.kb-dialog .el-textarea__inner::placeholder {
  color: #9CA3AF;
  font-weight: 300;
}
.kb-dialog .el-button {
  font-weight: 400;
}

/* 取消按钮 - 白底灰边 */
.kb-dialog .cancel-btn {
  --el-button-bg-color: #fff;
  --el-button-border-color: #d0d0cf;
  --el-button-text-color: #1A1A1A;
  --el-button-hover-bg-color: #f7f7f7;
  --el-button-hover-border-color: #9B9A97;
  --el-button-hover-text-color: #1A1A1A;
  --el-button-active-bg-color: #f0efed;
  --el-button-active-border-color: #9B9A97;
  --el-button-active-text-color: #1A1A1A;
  border-radius: 4px;
  font-weight: 400;
}

/* 确定按钮 - 黑底白字 */
.kb-dialog .confirm-btn {
  --el-button-bg-color: #1A1A1A;
  --el-button-border-color: #1A1A1A;
  --el-button-text-color: #fff;
  --el-button-hover-bg-color: #37352F;
  --el-button-hover-border-color: #37352F;
  --el-button-hover-text-color: #fff;
  --el-button-active-bg-color: #2c2a26;
  --el-button-active-border-color: #2c2a26;
  --el-button-active-text-color: #fff;
  border-radius: 4px;
  font-weight: 400;
}



/* 分页下拉框 */
.el-select-dropdown__item.selected {
  color: #37352F;
  font-weight: 500;
}
.el-select-dropdown__item.hover {
  background-color: #f0efed;
}

/* 弹出确认按钮 */
.el-popconfirm .el-button--primary {
  --el-button-bg-color: #37352F;
  --el-button-border-color: #37352F;
  --el-button-text-color: #fff;
  --el-button-hover-bg-color: #4a4842;
  --el-button-hover-border-color: #4a4842;
  --el-button-active-bg-color: #2c2a26;
  --el-button-active-border-color: #2c2a26;
}

/* 分页页码不加粗 */
.el-pagination .el-pager li.is-active {
  font-weight: 400;
}
</style>
