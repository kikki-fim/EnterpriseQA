<template>
  <!-- 文档管理页面 -->
  <div class="page-container">

    <!-- 操作栏 -->
    <el-card shadow="never" class="search-bar">
      <el-row :gutter="16" align="middle">
        <el-col :span="8">
          <el-select
            v-model="queryParams.kb_id"
            placeholder="选择知识库筛选"
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
        <el-col :span="16" style="text-align: right">
          <div class="btn-wrapper">
            <el-button class="add-btn" :icon="Upload" @click="uploadVisible = true">上传文档</el-button>
            <div class="btn-en">Upload Document</div>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <!-- 数据表格 -->
    <el-card shadow="never" class="table-card">
      <el-table :data="tableData" v-loading="loading" stripe>
        <el-table-column prop="id" label="ID" width="60" />
        <el-table-column prop="file_name" label="文件名" min-width="200" show-overflow-tooltip />
        <el-table-column prop="kb_name" label="所属知识库" width="150" />
        <el-table-column prop="file_type" label="类型" width="80" align="center">
          <template #default="{ row }">
            <el-tag :style="getFileTypeStyle(row.file_type)" size="small">
              {{ row.file_type }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="大小" width="100" align="center">
          <template #default="{ row }">
            {{ formatSize(row.file_size) }}
          </template>
        </el-table-column>
        <el-table-column prop="chunk_count" label="分块数" width="80" align="center" />
        <el-table-column prop="status" label="状态" width="100" align="center">
          <template #default="{ row }">
            <el-tag :type="statusMap[row.status]?.type" size="small">
              {{ statusMap[row.status]?.label }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="create_time" label="上传时间" width="170" />
        <el-table-column label="操作" width="80" fixed="right">
          <template #default="{ row }">
            <el-popconfirm title="确认删除该文档？" @confirm="handleDelete(row.id)">
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

    <!-- 上传对话框 -->
    <el-dialog
      v-model="uploadVisible"
      class="kb-dialog"
      title="上传文档"
      width="500px"
    >
      <el-form label-width="100px">
        <el-form-item label="选择知识库" required>
          <el-select v-model="uploadKbId" placeholder="请选择知识库" style="width: 100%">
            <el-option
              v-for="kb in kbOptions"
              :key="kb.id"
              :label="kb.kb_name"
              :value="kb.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="选择文件" required>
          <el-upload
            ref="uploadRef"
            v-model:file-list="fileList"
            :auto-upload="false"
            :limit="1"
            :on-exceed="() => ElMessage.warning('只能上传一个文件')"
            accept=".txt,.pdf,.md,.docx"
          >
            <el-button class="dialog-btn">选择文件</el-button>
            <template #tip>
              <div class="el-upload__tip">支持 txt、pdf、md、docx 格式，最大50MB</div>
            </template>
          </el-upload>
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
 * 文档管理页面
 * 支持按知识库筛选文档、上传新文档和删除文档
 */
import { ref, reactive, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { Upload } from '@element-plus/icons-vue'
import { getDocList, uploadDoc, deleteDoc } from '../api/document'
import { getAllKB } from '../api/knowledge'



const loading = ref(false)
const uploading = ref(false)
const uploadVisible = ref(false)
const tableData = ref([])
const total = ref(0)
const kbOptions = ref([])
const uploadKbId = ref(null)
const uploadRef = ref(null)
const fileList = ref([])

/** 查询参数 */
const queryParams = reactive({ page: 1, page_size: 10, kb_id: null })

/** 总页数 */
const totalPages = computed(() => {
  return Math.max(1, Math.ceil(total.value / queryParams.page_size))
})

/** 文档状态映射 */
const statusMap = {
  uploading: { label: '处理中', type: 'warning' },
  vectorized: { label: '已就绪', type: 'success' },
  failed: { label: '失败', type: 'danger' }
}

/** 文件格式 → 标签颜色映射 */
const fileTypeColor = {
  pdf: { bg: '#fdecec', color: '#d03050' },
  docx: { bg: '#eaf2ff', color: '#2f6fd0' },
  md: { bg: '#e8f8ee', color: '#2e9e5b' },
  txt: { bg: '#fdf3e3', color: '#d9822b' }
}

/** 根据文件格式返回标签样式 */
function getFileTypeStyle(fileType) {
  const style = fileTypeColor[fileType]
  if (!style) return {}
  return {
    backgroundColor: style.bg,
    color: style.color,
    borderColor: style.bg
  }
}

/** 格式化文件大小 */
function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

/** 加载知识库下拉选项 */
async function loadKBOptions() {
  const res = await getAllKB()
  kbOptions.value = res.data
}

/** 加载文档列表 */
async function loadList() {
  loading.value = true
  try {
    const res = await getDocList(queryParams)
    tableData.value = res.data.list
    total.value = res.data.total
  } finally {
    loading.value = false
  }
}

/** 处理文档上传 */
async function handleUpload() {
  if (!uploadKbId.value) {
    return ElMessage.warning('请选择知识库')
  }
  if (fileList.value.length === 0) {
    return ElMessage.warning('请选择文件')
  }

  const formData = new FormData()
  formData.append('file', fileList.value[0].raw)
  formData.append('kb_id', uploadKbId.value)

  uploading.value = true
  try {
    await uploadDoc(formData)
    ElMessage.success('上传成功')
    uploadVisible.value = false
    fileList.value = []
    loadList()
  } finally {
    uploading.value = false
  }
}

/** 删除文档 */
async function handleDelete(id) {
  await deleteDoc(id)
  ElMessage.success('删除成功')
  loadList()
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

/* 删除按钮颜色 */
.table-card :deep(.el-button--danger.is-link) {
  color: #ff8585;
}
.table-card :deep(.el-button--danger.is-link:hover) {
  color: #b53c3c;
}

/* 状态标签圆角 */
.table-card :deep(.el-tag) {
  border-radius: 2px;
}


.pagination :deep(.el-pagination) {
  color: #1A1A1A;
  font-weight: 400;
  font-size: 12px;
}

/* 上传提示文字 */
.search-bar :deep(.el-upload__tip) {
  color: #9CA3AF;
  font-weight: 300;
  font-size: 11px;
  line-height: 1.5;
  margin-top: 8px;
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
.kb-dialog .el-input__wrapper,
.kb-dialog .el-select .el-input__wrapper {
  background: #fff;
  box-shadow: 0 0 0 1px #d0d0cf inset;
  border-radius: 6px;
}
.kb-dialog .el-input__wrapper:hover,
.kb-dialog .el-select .el-input__wrapper:hover {
  box-shadow: 0 0 0 1px #9B9A97 inset;
}
.kb-dialog .el-input__wrapper.is-focus,
.kb-dialog .el-select .el-input__wrapper.is-focus {
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
.kb-dialog .el-upload__tip {
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



/* 下拉框选项（分页 + 筛选 + 对话框共用） */
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
