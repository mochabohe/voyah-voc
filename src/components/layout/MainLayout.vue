<template>
  <div class="layout-container">
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px' : '220px'" class="sidebar">
      <div class="logo-container">
        <template v-if="!isCollapse">
          <img src="/src/assets/logo.svg" alt="Voyah Logo" class="logo-image" />
          <h2>VOC分析平台</h2>
        </template>
        <h2 v-else>V</h2>
      </div>
      <el-menu
        :default-active="activeMenu"
        class="sidebar-menu"
        :collapse="isCollapse"
        background-color="#304156"
        text-color="#bfcbd9"
        active-text-color="#409EFF"
      >
        <!-- 声量智能探测（无详情时） -->
        <el-menu-item index="0" @click="currentView = 'home'" v-if="!showVoiceDetectionDetail">
          <el-icon><Microphone /></el-icon>
          <template #title>声量智能探测</template>
        </el-menu-item>
        
        <!-- 声量智能探测（有详情时） -->
        <el-sub-menu index="0" v-else @click="currentView = 'home'">
          <template #title>
            <el-icon><Microphone /></el-icon>
            <span>声量智能探测</span>
          </template>
          <el-menu-item index="0-1" @click="currentView = 'voice-detection'">
            <el-icon><DataAnalysis /></el-icon>
            <template #title>{{ selectedRequirement }}</template>
          </el-menu-item>
        </el-sub-menu>

        <!-- 全场景VOC分析 -->
        <el-sub-menu index="1">
          <template #title>
            <el-icon><DataAnalysis /></el-icon>
            <span>全场景VOC分析</span>
          </template>
          <el-menu-item index="1-1" @click="currentView = 'analysis'">
            <el-icon><DataBoard /></el-icon>
            <template #title>智能分析</template>
          </el-menu-item>
          <el-menu-item index="1-2" @click="currentView = 'monitoring'">
            <el-icon><Monitor /></el-icon>
            <template #title>监控预警</template>
          </el-menu-item>
        </el-sub-menu>

        <!-- 企划需求评估 -->
        <el-sub-menu index="2">
          <template #title>
            <el-icon><Document /></el-icon>
            <span>企划需求评估</span>
          </template>
          <el-menu-item index="2-1" @click="currentView = 'competitor'">
            <el-icon><TrendCharts /></el-icon>
            <template #title>竞品对比分析</template>
          </el-menu-item>
          <el-menu-item index="2-2" @click="currentView = 'feasibility'">
            <el-icon><Notebook /></el-icon>
            <template #title>可行性分析</template>
          </el-menu-item>
        </el-sub-menu>
      </el-menu>
    </el-aside>

    <!-- 主内容区 -->
    <el-container class="main-container">
      <!-- 顶部栏 -->
      <el-header class="header">
        <div class="header-left">
          <el-icon class="collapse-icon" @click="toggleCollapse">
            <Expand v-if="isCollapse" />
            <Fold v-else />
          </el-icon>
          <span class="page-title">{{ pageTitle }}</span>
        </div>
        <div class="header-right">
          <span class="user-info">岚图汽车 VOC 分析系统</span>
        </div>
      </el-header>

      <!-- 主体内容 -->
      <el-main class="content">
        <slot :currentView="currentView"></slot>
      </el-main>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";

const isCollapse = ref(false);
const currentView = ref("home");
const showVoiceDetectionDetail = ref(false);
const selectedRequirement = ref("");

// 监听localStorage变化
const checkSelectedRequirement = () => {
  const requirement = localStorage.getItem('selectedRequirement');
  if (requirement) {
    selectedRequirement.value = requirement;
    // 只有在voice-detection页面时才显示详情导航
    if (currentView.value === 'voice-detection') {
      showVoiceDetectionDetail.value = true;
    }
  } else {
    showVoiceDetectionDetail.value = false;
  }
};

// 初始检查
checkSelectedRequirement();

// 监听currentView变化
watch(currentView, (newView) => {
  if (newView === 'voice-detection') {
    // 进入详情页时显示详情导航
    checkSelectedRequirement();
  } else {
    // 离开详情页时隐藏详情导航
    showVoiceDetectionDetail.value = false;
  }
});

const activeMenu = computed(() => {
  const map: Record<string, string> = {
    home: "0",
    analysis: "1-1",
    monitoring: "1-2",
    "voice-detection": "0-1",
    competitor: "2-1",
    feasibility: "2-2",
  };
  return map[currentView.value] || "0";
});

const pageTitle = computed(() => {
  const titles: Record<string, string> = {
    home: "声量智能探测 - 热门需求",
    analysis: "全场景VOC分析 - 智能分析",
    monitoring: "全场景VOC分析 - 监控预警",
    "voice-detection": `声量智能探测 - ${selectedRequirement.value || '详情'}`,
    competitor: "企划需求评估 - 竞品对比分析",
    feasibility: "企划需求评估 - 可行性分析",
  };
  return titles[currentView.value] || "VOC分析平台";
});

const toggleCollapse = () => {
  isCollapse.value = !isCollapse.value;
};

defineExpose({ currentView });
</script>

<style scoped>
.layout-container {
  display: flex;
  height: 100vh;
  overflow: hidden;
}

.sidebar {
  background-color: #304156;
  overflow-x: hidden;
  transition: width 0.3s;
}

.logo-container {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #2b3a4a;
  color: #fff;
  font-weight: bold;
  font-size: 18px;
  border-bottom: 1px solid #1f2d3d;
  padding: 0 12px;
}

.logo-image {
  height: 28px;
  width: auto;
  max-width: 60px;
  margin-right: 8px;
  /* 移除滤镜，显示原始logo颜色 */
}

.logo-container h2 {
  margin: 0;
  font-size: 16px;
  white-space: nowrap;
}

.sidebar-menu {
  border-right: none;
  height: calc(100vh - 60px);
}

.main-container {
  flex: 1;
  background-color: #f0f2f5;
}

.header {
  background-color: #fff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.collapse-icon {
  font-size: 20px;
  cursor: pointer;
  transition: color 0.3s;
}

.collapse-icon:hover {
  color: #409eff;
}

.page-title {
  font-size: 16px;
  font-weight: 500;
  color: #303133;
}

.user-info {
  color: #606266;
  font-size: 14px;
}

.content {
  padding: 20px;
  overflow-y: auto;
  background-color: #f0f2f5;
}
</style>
