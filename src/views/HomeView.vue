<template>
  <div class="home-page">
    <!-- 热门问题卡片 -->
    <el-card class="hot-issues-card" shadow="hover">
      <template #header>
        <div class="card-header">
          <el-icon><TrendCharts /></el-icon>
          <span>热门问题</span>
          <span style="font-size: 12px; color: #909399; margin-left: auto;">基于全部用户反馈</span>
        </div>
      </template>
      
      <el-tabs v-model="activeTab" type="card">
        <!-- 全部 -->
        <el-tab-pane label="全部" name="all">
          <div class="hot-issues-grid">
            <div
              v-for="(issue, index) in mockData.hotIssues.all"
              :key="index"
              class="hot-issue-item"
              @click="handleIssueClick(issue)"
            >
              <div class="issue-name">{{ issue.name }}</div>
              <div class="issue-stats">
                <span class="issue-count">{{ issue.count }}次</span>
                <span
                  class="issue-trend"
                  :class="{ positive: issue.trend.includes('+') }"
                >
                  {{ issue.trend }}
                </span>
              </div>
              <div class="issue-description">{{ issue.description }}</div>
            </div>
          </div>
        </el-tab-pane>

        <!-- 公域 -->
        <el-tab-pane label="公域" name="public">
          <div class="hot-issues-grid">
            <div
              v-for="(issue, index) in mockData.hotIssues.public"
              :key="index"
              class="hot-issue-item"
              @click="handleIssueClick(issue)"
            >
              <div class="issue-name">{{ issue.name }}</div>
              <div class="issue-stats">
                <span class="issue-count">{{ issue.count }}次</span>
                <span
                  class="issue-trend"
                  :class="{ positive: issue.trend.includes('+') }"
                >
                  {{ issue.trend }}
                </span>
              </div>
              <div class="issue-description">{{ issue.description }}</div>
            </div>
          </div>
        </el-tab-pane>

        <!-- 私域 -->
        <el-tab-pane label="私域" name="private">
          <div class="hot-issues-grid">
            <div
              v-for="(issue, index) in mockData.hotIssues.private"
              :key="index"
              class="hot-issue-item"
              @click="handleIssueClick(issue)"
            >
              <div class="issue-name">{{ issue.name }}</div>
              <div class="issue-stats">
                <span class="issue-count">{{ issue.count }}次</span>
                <span
                  class="issue-trend"
                  :class="{ positive: issue.trend.includes('+') }"
                >
                  {{ issue.trend }}
                </span>
              </div>
              <div class="issue-description">{{ issue.description }}</div>
            </div>
          </div>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  <div class="hot-requirements-section">
    <div class="req-header-wrapper">
      <h3 class="req-main-title">热门需求</h3>
      
      <div class="filter-controls">
        <div class="filters">
          <span>筛选条件：</span>
          <el-tag 
            v-for="filter in requirementFilters" 
            :key="filter.label"
            :effect="activeRequirementFilter === filter.label ? 'dark' : 'plain'"
            :type="activeRequirementFilter === filter.label ? 'primary' : 'info'"
            class="filter-tag"
            @click="activeRequirementFilter = filter.label"
          >
            {{ filter.label }}
          </el-tag>
          <el-icon class="chart-icon"><DataAnalysis /></el-icon>
        </div>
        <div class="sorts">
          <span>排序：</span>
          <span class="sort-item">机会评分 <el-icon><Bottom /></el-icon></span>
          <span class="sort-item">用户声量 <el-icon><Bottom /></el-icon></span>
          <span class="sort-item">技术成熟度 <el-icon><Bottom /></el-icon></span>
          <span class="sort-item">竞品覆盖 <el-icon><Bottom /></el-icon></span>
        </div>
      </div>
    </div>

    <div class="realtime-notice">
      [实时发现] 有5个新机会
    </div>

    <div class="emergency-section-wrapper">
      <div class="emergency-section">
        <div class="section-title">
          <el-icon color="white"><Bell /></el-icon> 紧急机会（评分>85）
        </div>
        
        <div class="opportunity-cards">
          <!-- Card 1 -->
          <div class="opp-card" @click="handleCardClick('方向盘加热')">
            <div class="opp-header">
              <el-icon color="#FF6B6B" :size="20"><Histogram /></el-icon> 
              <span class="opp-title">方向盘加热</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>92</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <span class="voice-icons">
                  <el-icon color="#FFD700"><Sunny /></el-icon>
                  <el-icon color="#FFD700"><Sunny /></el-icon>
                  <el-icon color="#FFD700"><Sunny /></el-icon>
                </span>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#90EE90"><Check /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>4/5</span>
              </div>
            </div>
          </div>

          <!-- Card 2 -->
          <div class="opp-card" @click="handleCardClick('副驾娱乐屏')">
            <div class="opp-header">
              <el-icon color="#4FC3F7" :size="20"><Monitor /></el-icon> 
              <span class="opp-title">副驾娱乐屏</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>88</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <span class="voice-icons">
                  <el-icon color="#FFD700"><Sunny /></el-icon>
                  <el-icon color="#FFD700"><Sunny /></el-icon>
                </span>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#90EE90"><Check /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>3/5</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 高价值机会 -->
    <div class="value-section-wrapper">
      <div class="value-section">
        <div class="section-title">
          <el-icon color="white"><Lightbulb /></el-icon> 高价值机会（评分70-85）
        </div>
        
        <div class="opportunity-cards">
          <!-- K歌音效 -->
          <div class="opp-card" @click="handleCardClick('K歌音效')">
            <div class="opp-header">
              <el-icon color="#FFB74D" :size="20"><Microphone /></el-icon> 
              <span class="opp-title">K歌音效</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>82</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <el-icon color="#FF6B6B" :size="18"><HomeFilled /></el-icon>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#90EE90"><Check /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>3/5</span>
              </div>
            </div>
          </div>

          <!-- 车载冰箱 -->
          <div class="opp-card" @click="handleCardClick('车载冰箱')">
            <div class="opp-header">
              <el-icon color="#81C784" :size="20"><ColdDrink /></el-icon> 
              <span class="opp-title">车载冰箱</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>78</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <el-icon color="#FF6B6B" :size="18"><HomeFilled /></el-icon>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#FFB74D"><Warning /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>2/5</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 潜力机会 -->
    <div class="potential-section-wrapper">
      <div class="potential-section">
        <div class="section-title">
          <el-icon color="white"><MagicStick /></el-icon> 潜力机会（评分60-70）
        </div>
        
        <div class="opportunity-cards">
          <!-- 声纹识别 -->
          <div class="opp-card" @click="handleCardClick('声纹识别')">
            <div class="opp-header">
              <el-icon color="#BA68C8" :size="20"><Microphone /></el-icon> 
              <span class="opp-title">声纹识别</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>68</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <el-icon color="#FFB74D" :size="18"><Warning /></el-icon>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#FFB74D"><Warning /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>1/5</span>
              </div>
            </div>
          </div>

          <!-- 哨兵模式 -->
          <div class="opp-card" @click="handleCardClick('哨兵模式')">
            <div class="opp-header">
              <el-icon color="#64B5F6" :size="20"><VideoCamera /></el-icon> 
              <span class="opp-title">哨兵模式</span>
            </div>
            <div class="opp-content">
              <div class="score-row">评分：<strong>65</strong></div>
              <div class="detail-row">
                <span>用户声量：</span>
                <el-icon color="#FFB74D" :size="18"><Warning /></el-icon>
              </div>
              <div class="detail-row">
                <span>技术：</span>
                <el-icon color="#90EE90"><Check /></el-icon>
              </div>
              <div class="detail-row">
                <span>竞品：</span>
                <span>4/5</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="trend-section-wrapper">
      <div class="trend-section">
        <div class="section-title trend-title">
          需求来源趋势（过去30天）
        </div>
        <div class="trend-box">
          <div class="trend-item">
            <span class="label">用户来源：</span>
            <span class="value">APP社区 45%  客服反馈 35%  售后 20%</span>
          </div>
          <div class="trend-item">
            <span class="label">高频需求：</span>
            <span class="value">1.座椅舒适性  2.娱乐系统  3.储物空间</span>
          </div>
          <div class="trend-item">
            <span class="label">增长最快：</span>
            <span class="value">露营相关需求 (+120%)</span>
          </div>
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import analysisData from '../mock/analysis.json';

const emit = defineEmits<{
  navigate: [view: string]
}>();

const activeTab = ref('all');
const mockData = ref(analysisData);

const activeRequirementFilter = ref('全部');
const requirementFilters = ref([
  { label: '全部', type: 'info', icon: '' },
  { label: '高优先级', type: 'plain', icon: 'Search' },
  { label: '技术可行', type: 'plain', icon: 'Sunny' },
  { label: '竞品验证', type: 'plain', icon: 'Check' }
]);

const handleCardClick = (requirementName: string) => {
  // 跳转到声量智能探测页面
  emit('navigate', 'voice-detection');
  // 可以在这里存储选中的需求名称到localStorage或store
  localStorage.setItem('selectedRequirement', requirementName);
};

// 处理热门问题卡片点击
const handleIssueClick = (issue: any) => {
  // 跳转到智能分析页面
  emit('navigate', 'analysis');
  // 存储选中的问题信息到localStorage，供分析页面使用
  localStorage.setItem('selectedIssue', JSON.stringify({
    name: issue.name,
    count: issue.count,
    trend: issue.trend,
    description: issue.description
  }));
};
</script>

<style scoped>
    .home-page {
      padding: 24px;
      min-height: 100%;
      background: #f5f7fa;
    }
    
    .hot-issues-card {
      margin-bottom: 20px;
    }

    .hot-requirements-section {
      margin-top: 20px;
    }

    .req-header-wrapper {
      background: white;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 16px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
    }

    .req-main-title {
      font-size: 18px;
      font-weight: 600;
      color: #303133;
      margin: 0 0 16px 0;
    }

    .filter-controls {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .filters {
      display: flex;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
    }

    .filters > span {
      font-size: 14px;
      color: #606266;
      font-weight: 500;
    }

    .filter-tag {
      cursor: pointer;
      font-size: 13px;
    }

    .chart-icon {
      font-size: 18px;
      color: #409EFF;
      cursor: pointer;
      margin-left: 8px;
    }

    .sorts {
      display: flex;
      align-items: center;
      gap: 16px;
      font-size: 13px;
      color: #909399;
      flex-wrap: wrap;
    }

    .sorts > span:first-child {
      font-weight: 500;
      color: #606266;
    }

    .sort-item {
      display: inline-flex;
      align-items: center;
      gap: 4px;
      cursor: pointer;
      transition: color 0.3s;
    }

    .sort-item:hover {
      color: #409EFF;
    }

    .sort-item .el-icon {
      font-size: 12px;
    }


    .realtime-notice {
      margin-bottom: 20px;
      font-size: 14px;
      color: #303133;
      font-weight: 500;
    }

    .emergency-section-wrapper {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      border-radius: 8px;
      padding: 24px;
      margin-bottom: 20px;
    }

    .value-section-wrapper {
      background: linear-gradient(135deg, #FFA726 0%, #FB8C00 100%);
      border-radius: 8px;
      padding: 24px;
      margin-bottom: 20px;
    }

    .potential-section-wrapper {
      background: linear-gradient(135deg, #26A69A 0%, #00897B 100%);
      border-radius: 8px;
      padding: 24px;
      margin-bottom: 20px;
    }

    .section-title {
      font-weight: bold;
      margin-bottom: 16px;
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 15px;
      color: white;
    }

    .opportunity-cards {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 16px;
      margin-bottom: 24px;
    }

    .opp-card {
      background: rgba(255, 255, 255, 0.15);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255, 255, 255, 0.2);
      border-radius: 6px;
      padding: 12px 16px;
      color: white;
      transition: all 0.3s;
      cursor: pointer;
    }

    .opp-card:hover {
      background: rgba(255, 255, 255, 0.25);
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
    }

    .opp-header {
      display: flex;
      align-items: center;
      gap: 6px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.2);
      padding-bottom: 8px;
      margin-bottom: 8px;
      font-weight: bold;
      font-size: 14px;
    }

    .opp-content .score-row {
      margin-bottom: 8px;
      font-size: 12px;
    }

    .opp-content .score-row strong {
      font-size: 20px;
      margin-left: 4px;
    }

    .opp-content .detail-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 6px;
      font-size: 12px;
      opacity: 0.95;
    }

    .voice-icons {
      display: inline-flex;
      align-items: center;
      gap: 2px;
    }

    .voice-icons .el-icon {
      font-size: 12px;
    }

    .dash-lines {
      display: none;
    }

    .dash-line {
      display: none;
    }

    .trend-box {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      border-radius: 0 0 8px 8px;
      padding: 20px;
      color: white;
      border: none;
    }

    .trend-title {
      color: #303133 !important;
      background: white;
      padding: 12px 16px;
      border-radius: 8px 8px 0 0;
      margin-bottom: 0 !important;
    }

    .trend-item {
      margin-bottom: 12px;
      font-size: 14px;
      line-height: 1.6;
    }
    .trend-item:last-child {
      margin-bottom: 0;
    }

    .trend-item .label {
      color: rgba(255, 255, 255, 0.9);
      display: inline-block;
      width: 100px;
      font-weight: 500;
    }
    
    .trend-item .value {
      color: white;
      font-weight: 500;
    }

    /* Reuse existing styles */
    .hot-issues-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 16px;
      padding: 16px 0;
    }
    
    .hot-issue-item {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      padding: 20px;
      border-radius: 8px;
      color: white;
      transition: all 0.3s;
      cursor: pointer;
    }
    
    .hot-issue-item:hover {
      transform: translateY(-4px);
      box-shadow: 0 8px 16px rgba(102, 126, 234, 0.3);
    }
    
    .issue-name {
      font-size: 18px;
      font-weight: bold;
      margin-bottom: 12px;
    }
    
    .issue-stats {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 12px;
    }
    
    .issue-count {
      font-size: 24px;
      font-weight: bold;
    }
    
    .issue-trend {
      font-size: 14px;
      padding: 2px 8px;
      border-radius: 4px;
      background: rgba(255, 255, 255, 0.2);
    }
    
    .issue-trend.positive {
      background: rgba(103, 194, 58, 0.3);
    }
    
    .issue-description {
      font-size: 13px;
      opacity: 0.9;
      line-height: 1.5;
    }
  </style>
