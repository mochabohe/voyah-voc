<template>
  <div class="analysis-page">
    <!-- 筛选条件 -->
    <el-card class="filter-card" shadow="never" style="margin-top: 20px">
      <template #header>
        <div class="card-header">
          <el-icon><Filter /></el-icon>
          <span>筛选条件</span>
        </div>
      </template>
      <el-form :inline="true">
        <el-form-item label="时间范围">
          <el-date-picker
            v-model="dateRange"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            size="default"
          />
        </el-form-item>
        <el-form-item label="车型">
          <el-select
            v-model="selectedModel"
            placeholder="请选择车型"
            size="default"
          >
            <el-option label="全部类型" value="" />
            <el-option label="H56D" value="H56D" />
            <el-option label="H77A" value="H77A" />
            <el-option label="H37A" value="H37A" />
          </el-select>
        </el-form-item>
        <el-form-item label="问题类型">
          <el-select
            v-model="selectedType"
            placeholder="全部类型"
            size="default"
            style="width: 200px"
          >
            <el-option label="全部类型" value="" />
            <el-option label="硬件故障" value="硬件故障" />
            <el-option label="软件故障" value="软件故障" />
            <el-option label="新增功能" value="新增功能" />
            <el-option label="新增配置" value="新增配置" />
            <el-option label="硬件优化" value="硬件优化" />
            <el-option label="功能优化" value="功能优化" />
          </el-select>
        </el-form-item>
        <el-form-item label="部件">
          <el-input
            v-model="selectedComponent"
            placeholder="输入框，用户在里面输入，可以用'摄像头、语音交互、氛围灯'"
            clearable
            size="default"
            style="width: 280px"
          >
            <template #prefix>
              <el-icon><Search /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item label="客户情绪">
          <el-checkbox-group v-model="selectedEmotions">
            <el-checkbox label="正面" />
            <el-checkbox label="负面" />
            <el-checkbox label="中性" />
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="反馈来源">
          <el-checkbox-group v-model="selectedSources">
            <el-checkbox label="CC" />
            <el-checkbox label="APP" />
            <el-checkbox label="质检" />
            <el-checkbox label="小红书" />
            <el-checkbox label="抖音" />
          </el-checkbox-group>
        </el-form-item>
        <el-form-item>
          <el-button @click="resetFilters">
            <el-icon><RefreshRight /></el-icon>
            重置
          </el-button>
          <el-button type="primary" @click="applyFilters">
            <el-icon><DataAnalysis /></el-icon>
            分析
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 问题列表 -->
    <el-card class="table-card" shadow="hover" style="margin-top: 20px">
      <template #header>
        <div class="card-header">
          <div>
            <el-icon><List /></el-icon>
            <span>问题列表</span>
          </div>
          <div>
            <span style="font-size: 14px; color: #909399">
              反馈总数：<span style="color: #409eff; font-weight: bold">{{
                filteredIssues.length
              }}</span>
            </span>
          </div>
        </div>
      </template>
      <el-table :data="filteredIssues" stripe style="width: 100%">
        <el-table-column prop="id" label="用户ID" width="100" />
        <el-table-column prop="content" label="问题描述" min-width="300" />
        <el-table-column prop="model" label="车型" width="100">
          <template #default="{ row }">
            <el-tag>{{ row.model }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="type" label="问题类型" width="120">
          <template #default="{ row }">
            <el-tag :type="getTypeColor(row.type)">{{ row.type }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="emotion" label="情感" width="100">
          <template #default="{ row }">
            <el-tag :type="getEmotionColor(row.emotion)">{{
              row.emotion
            }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="source" label="反馈来源" width="120" />
        <el-table-column prop="time" label="反馈时间" width="120" />
      </el-table>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, onMounted } from "vue";
import * as echarts from "echarts";
import type { ECharts } from "echarts";
import analysisData from "../mock/analysis.json";

// 筛选条件
const dateRange = ref<[Date, Date] | null>(null);
const selectedModel = ref("");
const selectedType = ref("");
const selectedComponent = ref("");
const selectedEmotions = ref<string[]>(["正面", "负面", "中性"]);
const selectedSources = ref<string[]>([
  "CC",
  "APP",
  "质检",
  "小红书",
  "抖音",
]);

// Mock 数据
const mockData = ref(analysisData);

// 图表实例
const emotionChart = ref<HTMLElement>();
let emotionChartInstance: ECharts | null = null;

// 过滤后的问题列表
const filteredIssues = computed(() => {
  return mockData.value.issueList.filter((issue) => {
    const emotionMatch = selectedEmotions.value.includes(issue.emotion);
    const sourceMatch = selectedSources.value.includes(issue.source);
    // 使用 includes 进行模糊匹配，解决 H77 和 H77A 的匹配问题
    const modelMatch =
      !selectedModel.value || issue.model.includes(selectedModel.value);
    return emotionMatch && sourceMatch && modelMatch;
  });
});

const applyFilters = () => {
  // 应用筛选条件，触发 computedFilter
  console.log("应用筛选条件");
};

const resetFilters = () => {
  dateRange.value = null;
  selectedModel.value = "";
  selectedType.value = "";
  selectedComponent.value = "";
  selectedEmotions.value = ["正面", "负面", "中性"];
  selectedSources.value = ["CC", "APP", "质检", "小红书", "抖音"];
};

const initCharts = () => {
  if (emotionChart.value) {
    emotionChartInstance = echarts.init(emotionChart.value);
    initEmotionChart();
  }
};

// 页面加载时初始化图表和处理传入的问题
onMounted(() => {
  // 检查是否有从首页传来的问题信息
  const selectedIssueStr = localStorage.getItem('selectedIssue');
  if (selectedIssueStr) {
    try {
      const selectedIssue = JSON.parse(selectedIssueStr);
      console.log('接收到从首页传来的问题:', selectedIssue);
      
      // 特殊处理：如果是H77扶手屏问题，直接展示所有相关数据（忽略正则提取）
      // 满足用户需求：显示全部mock数据，不使用简单正则匹配
      if (selectedIssue.name.includes('H77') || selectedIssue.name.includes('扶手屏')) {
        selectedModel.value = 'H77A'; // 强制匹配 H77A
        selectedComponent.value = ''; // 不预填部件名，避免误以为在进行关键词过滤
      } else {
        // 原有逻辑：正则提取
        const modelMatch = selectedIssue.name.match(/^(H\d+[A-Z]?)/);
        if (modelMatch) {
          selectedModel.value = modelMatch[1];
        }
        
        // 将问题名称设置到部件输入框
        const componentName = selectedIssue.name.replace(/^H\d+[A-Z]?/, ''); // 去掉车型前缀
        selectedComponent.value = componentName;
      }
      
      // 清除 localStorage 中的数据，避免重复使用
      localStorage.removeItem('selectedIssue');
      
      // 触发筛选
      applyFilters();
    } catch (e) {
      console.error('解析问题信息失败:', e);
    }
  }
  
  nextTick(() => {
    initCharts();
  });
});

const initEmotionChart = () => {
  if (!emotionChartInstance) return;

  const option = {
    tooltip: {
      trigger: "item",
      formatter: "{b}: {c}% ({d}%)",
    },
    legend: {
      orient: "vertical",
      left: "left",
      top: "center",
    },
    series: [
      {
        type: "pie",
        radius: ["40%", "70%"],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 10,
          borderColor: "#fff",
          borderWidth: 2,
        },
        label: {
          show: false,
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 16,
            fontWeight: "bold",
          },
        },
        data: mockData.value.emotionData.map((item) => ({
          value: item.value,
          name: item.name,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  emotionChartInstance.setOption(option);
};

const getTypeColor = (type: string) => {
  const colors: Record<string, string> = {
    硬件故障: "danger",
    软件故障: "warning",
    新增功能: "success",
    功能优化: "info",
  };
  return colors[type] || "";
};

const getEmotionColor = (emotion: string) => {
  const colors: Record<string, string> = {
    正面: "success",
    负面: "danger",
    中性: "info",
  };
  return colors[emotion] || "";
};

// 窗口大小变化时重绘图表
window.addEventListener("resize", () => {
  emotionChartInstance?.resize();
});
</script>

<style scoped>
.analysis-page {
  min-height: 100%;
}

.search-card {
  margin-bottom: 20px;
}

.search-input-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: #f5f7fa;
  border-radius: 8px;
}

.search-icon {
  font-size: 20px;
  color: #909399;
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  font-size: 16px;
  outline: none;
  color: #303133;
}

.search-input::placeholder {
  color: #c0c4cc;
}

.search-btn {
  padding: 12px 24px;
}

.search-tips {
  margin-top: 10px;
  font-size: 13px;
  color: #909399;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100px 0;
  color: #409eff;
}

.loading-container p {
  margin-top: 20px;
  font-size: 16px;
}

.hot-issues-card {
  margin-bottom: 20px;
}

.hot-issues-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.hot-issue-item {
  padding: 15px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  color: #fff;
  cursor: pointer;
  transition: transform 0.3s;
}

.hot-issue-item:hover {
  transform: translateY(-5px);
}

.issue-name {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.issue-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.issue-count {
  font-size: 20px;
  font-weight: bold;
}

.issue-trend {
  font-size: 14px;
  padding: 2px 8px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4px;
}

.issue-trend.positive {
  background: rgba(103, 194, 58, 0.3);
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-weight: 500;
}

.card-header > div {
  display: flex;
  align-items: center;
  gap: 8px;
}

.chart-actions {
  display: flex;
  gap: 10px;
}

.chart-container {
  width: 100%;
  height: 400px;
}

.chart-container-small {
  width: 100%;
  height: 400px;
}

.filter-card :deep(.el-form-item) {
  margin-bottom: 15px;
}

.table-card {
  margin-bottom: 20px;
}

.stats-card {
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.stats-card :deep(.el-card__body) {
  padding: 0;
}

.stats-content {
  padding: 30px 20px;
  text-align: center;
}

.stats-label {
  font-size: 14px;
  opacity: 0.9;
  margin-bottom: 10px;
}

.stats-value {
  font-size: 48px;
  font-weight: bold;
  line-height: 1;
  margin: 10px 0;
}

.stats-trend {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  font-size: 14px;
  margin-top: 12px;
}

.stats-trend.positive {
  color: #67c23a;
}

.stats-period {
  opacity: 0.8;
  font-size: 12px;
}
</style>
