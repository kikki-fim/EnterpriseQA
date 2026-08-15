<template>
  <!-- 管理员首页 - 数据统计概览 -->
  <div class="home-container">
    <!-- KPI 卡片行：中文标签 / 数字 / 英文小字，全部居中 -->
    <el-row :gutter="20" class="stat-cards">
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-label">知识库总数</div>
          <div class="stat-value">{{ formatNum(stats.kb_count) }}</div>
          <div class="stat-en">Knowledge Bases</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-label">文档总数</div>
          <div class="stat-value">{{ formatNum(stats.doc_count) }}</div>
          <div class="stat-en">Total Documents</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-label">注册用户数</div>
          <div class="stat-value">{{ formatNum(stats.user_count) }}</div>
          <div class="stat-en">Registered Users</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-label">问答次数</div>
          <div class="stat-value">{{ formatNum(stats.today_chat_count) }}</div>
          <div class="stat-en">Q&amp;A Sessions</div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 图表区域 -->
    <el-row :gutter="20" class="chart-row">
      <el-col :span="14">
        <div class="panel">
          <div class="panel-title">问答趋势</div>
          <div ref="trendChartRef" class="chart-box"></div>
          <div class="panel-divider"></div>
          <div class="panel-sub">每日提问次数</div>
        </div>
      </el-col>
      <el-col :span="10">
        <div class="panel">
          <div class="panel-title">知识库文档分布</div>
          <div class="donut-area">
            <div class="donut-wrap">
              <div ref="pieChartRef" class="donut-box"></div>
              <div class="donut-center">
                <span class="dc-val">{{ formatNum(donutTotal) }}</span>
                <span class="dc-label">总文档</span>
              </div>
            </div>
            <div class="donut-legend">
              <div v-for="item in legendData" :key="item.name" class="legend-row">
                <span class="legend-dot" :style="{ background: item.color }"></span>
                <span class="legend-name">{{ item.name }}</span>
              </div>
            </div>
          </div>
          <div class="panel-divider"></div>
          <div class="panel-sub">各知识库文档数量占比</div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
/**
 * 管理员首页 - 数据统计概览
 * Notion 风格：卡片居中三行式 / 橙黄趋势线 / 蓝紫单色环形图
 * 字体统一不加粗，数字使用清秀字重
 */
import { ref, reactive, computed, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'
import { getOverview } from '../api/stats'

/** 环形图蓝紫色板：#9e9bff 最深，依次降透明度 */
const DONUT_PALETTE = [
  '#9e9bff',
  'rgba(158, 155, 255, 0.78)',
  'rgba(158, 155, 255, 0.55)',
  'rgba(158, 155, 255, 0.38)',
  'rgba(158, 155, 255, 0.25)',
  'rgba(158, 155, 255, 0.12)'
]

/** 统计数据 */
const stats = reactive({
  user_count: 0,
  kb_count: 0,
  doc_count: 0,
  today_chat_count: 0,
  trend_data: [],
  kb_doc_data: []
})

/** 图表DOM引用 */
const trendChartRef = ref(null)
const pieChartRef = ref(null)
let trendChart = null
let pieChart = null

/** 数字千分位格式化 */
function formatNum(n) {
  return Number(n || 0).toLocaleString('en-US')
}

/** 文档总数（环形图中心） */
const donutTotal = computed(() => {
  return (stats.kb_doc_data || []).reduce((sum, item) => sum + item.value, 0)
})

/** 环形图右侧图例（只显示名称，不带数字和百分比） */
const legendData = computed(() => {
  const raw = stats.kb_doc_data.length > 0 ? stats.kb_doc_data : []
  return raw.map((item, index) => ({
    name: item.name,
    color: DONUT_PALETTE[index % DONUT_PALETTE.length]
  }))
})

/** 加载统计数据 */
async function loadStats() {
  try {
    const res = await getOverview()
    Object.assign(stats, res.data)
    renderTrendChart()
    renderPieChart()
  } catch (err) {
    // 错误已在拦截器中处理
  }
}

/** 渲染近7天提问趋势折线图（#f7a868） */
function renderTrendChart() {
  if (!trendChartRef.value) return
  trendChart = echarts.init(trendChartRef.value)

  // 只取最近 7 天数据
  const trend = stats.trend_data.slice(-7)
  const dates = trend.map(item => item.date)
  const counts = trend.map(item => item.count)

  trendChart.setOption({
    tooltip: {
      trigger: 'axis',
      backgroundColor: '#fff',
      borderColor: '#E9E9E7',
      borderWidth: 1,
      padding: 10,
      textStyle: { color: '#37352F', fontSize: 12 },
      axisPointer: { type: 'line', lineStyle: { color: '#E9E9E7' } }
    },
    grid: { left: '3%', right: '4%', bottom: '3%', top: 10, containLabel: true },
    xAxis: {
      type: 'category',
      data: dates,
      boundaryGap: false,
      axisLine: { lineStyle: { color: '#E9E9E7' } },
      axisTick: { show: false },
      axisLabel: { color: '#9B9A97', fontSize: 11 },
      splitLine: { show: false }
    },
    yAxis: {
      type: 'value',
      minInterval: 1,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: { color: '#9B9A97', fontSize: 11 },
      splitLine: { show: false }
    },
    series: [{
      name: '提问次数',
      type: 'line',
      smooth: true,
      data: counts,
      symbol: 'circle',
      symbolSize: 7,
      lineStyle: { color: '#f7a868', width: 2 },
      itemStyle: { color: '#f7a868', borderColor: '#fff', borderWidth: 2 },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: 'rgba(247, 168, 104, 0.35)' },
          { offset: 1, color: 'rgba(247, 168, 104, 0.03)' }
        ])
      }
    }]
  })
}

/** 渲染知识库文档占比环形图（蓝紫单色、加粗环、中心文字居中） */
function renderPieChart() {
  if (!pieChartRef.value) return
  pieChart = echarts.init(pieChartRef.value)

  const data = stats.kb_doc_data.length > 0
    ? stats.kb_doc_data
    : [{ name: '暂无数据', value: 1 }]

  pieChart.setOption({
    color: DONUT_PALETTE,
    tooltip: {
      trigger: 'item',
      backgroundColor: '#fff',
      borderColor: '#E9E9E7',
      borderWidth: 1,
      padding: 10,
      textStyle: { color: '#37352F', fontSize: 12 },
      formatter: '{b}: {c} 篇 ({d}%)'
    },
    legend: { show: false },
    series: [{
      type: 'pie',
      radius: ['70%', '100%'],
      center: ['50%', '50%'],
      avoidLabelOverlap: false,
      label: { show: false },
      itemStyle: { borderColor: '#fff', borderWidth: 3 },
      emphasis: { scaleSize: 4 },
      data: data
    }]
  })
}

/** 窗口大小变化时重绘图表 */
function handleResize() {
  trendChart?.resize()
  pieChart?.resize()
}

onMounted(() => {
  loadStats()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  trendChart?.dispose()
  pieChart?.dispose()
})
</script>

<style scoped>
.home-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* ---- KPI 卡片：居中三行式，放大版 ---- */
.stat-card {
  border: 1px solid #E8E8E8;
  border-radius: 12px;
  background: #fff;
  text-align: center;
}

.stat-card :deep(.el-card__body) {
  padding: 24px 28px;
}

.stat-label {
  font-size: 16px;
  color: #1A1A1A;
  font-weight: 400;
  text-align: center;
}

.stat-value {
  font-size: 40px;
  color: #111;
  font-weight: 300;
  line-height: 1.15;
  margin-top: 10px;
  letter-spacing: -0.5px;
  text-align: center;
  font-family: 'Inter', 'SF Pro Display', -apple-system, 'PingFang SC', 'Helvetica Neue', sans-serif;
}

.stat-en {
  font-size: 13px;
  color: #9CA3AF;
  font-weight: 400;
  margin-top: 8px;
  text-align: center;
}

/* ---- 图表面板 ---- */
.chart-row {
  margin-top: 0;
}

.panel {
  background: #fff;
  border: 1px solid #E8E8E8;
  border-radius: 12px;
  padding: 20px;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.panel-title {
  font-size: 15px;
  color: #37352F;
  font-weight: 400;
}

.panel-divider {
  height: 1px;
  background: #E8E8E8;
  margin: 12px 0 10px;
  flex-shrink: 0;
}

.panel-sub {
  font-size: 11px;
  color: #9B9A97;
  font-weight: 400;
  flex-shrink: 0;
}

/* ---- 趋势图 ---- */
.chart-box {
  flex: 1;
  min-height: 240px;
  margin-top: 12px;
}

/* ---- 环形图区域：左图 + 右侧图例 ---- */
.donut-area {
  display: flex;
  align-items: center;
  gap: 50px;
  margin-top: 19px;
  flex: 1;
}

/* 环形容器：定位中心文字 */
.donut-wrap {
  position: relative;
  width: 200px;
  height: 200px;
  flex-shrink: 0;
}

.donut-box {
  width: 100%;
  height: 100%;
}

.donut-center {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  pointer-events: none;
}

.dc-val {
  font-size: 22px;
  color: #37352F;
  font-weight: 300;
  line-height: 1;
  font-family: 'Inter', 'SF Pro Display', -apple-system, 'PingFang SC', 'Helvetica Neue', sans-serif;
}

.dc-label {
  font-size: 11px;
  color: #9B9A97;
  margin-top: 4px;
}

.donut-legend {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.legend-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
}

.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 3px;
  flex-shrink: 0;
}

.legend-name {
  flex: 1;
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #9B9A97;
}
</style>
