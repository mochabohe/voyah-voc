<script setup lang="ts">
import { ref } from "vue";
import MainLayout from "./components/layout/MainLayout.vue";
import HomeView from "./views/HomeView.vue";
import AnalysisView from "./views/AnalysisView.vue";
import FeasibilityView from "./views/FeasibilityView.vue";
import MonitoringView from "./views/MonitoringView.vue";
import VoiceDetectionView from "./views/VoiceDetectionView.vue";
import CompetitorView from "./views/CompetitorView.vue";
import ChatBot from "./components/ChatBot.vue";

const layoutRef = ref();

const handleNavigate = (view: string) => {
  if (layoutRef.value) {
    layoutRef.value.currentView = view;
  }
};
</script>

<template>
  <MainLayout ref="layoutRef">
    <template #default="{ currentView }">
      <HomeView v-if="currentView === 'home'" @navigate="handleNavigate" />
      <AnalysisView v-else-if="currentView === 'analysis'" />
      <MonitoringView v-else-if="currentView === 'monitoring'" />
      <FeasibilityView v-else-if="currentView === 'feasibility'" />
      <VoiceDetectionView v-else-if="currentView === 'voice-detection'" />
      <CompetitorView v-else-if="currentView === 'competitor'" />
      <div v-else class="placeholder-view">
        <el-empty description="功能开发中..." />
      </div>
    </template>
  </MainLayout>

  <!-- 悬浮聊天机器人 -->
  <ChatBot />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
    Arial, sans-serif;
}

#app {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.placeholder-view {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
</style>
