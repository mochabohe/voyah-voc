<template>
  <div class="competitor-page">
    <el-card class="header-card">
      <h2>竞品对比分析</h2>
      <p style="margin-top: 12px; color: #606266;">
        <strong>功能描述：</strong>输入具体需求功能点或车型，进行细致的竞品对比。
      </p>
    </el-card>

    <!-- 输入区域 -->
    <el-card style="margin-top: 16px;">
      <el-form :inline="true">
        <el-form-item label="输入功能点">
          <el-input
            v-model="featureInput"
            placeholder="如：智能座舱语音交互"
            style="width: 300px"
          />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="analyzeCompetitor">
            <el-icon><Search /></el-icon>
            对比分析
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 竞品功能对比矩阵 -->
    <el-card style="margin-top: 16px;">
      <template #header>
        <div style="font-weight: bold;">竞品功能对标矩阵</div>
      </template>
      <el-table :data="comparisonData" border style="width: 100%">
        <el-table-column prop="feature" label="功能点" width="150" fixed />
        <el-table-column label="岚图FREE" align="center">
          <el-table-column prop="lantu" label="支持情况" align="center">
            <template #default="scope">
              <el-tag v-if="scope.row.lantu === '支持'" type="success" size="small">
                ✓ {{ scope.row.lantu }}
              </el-tag>
              <el-tag v-else type="danger" size="small">✗ {{ scope.row.lantu }}</el-tag>
            </template>
          </el-table-column>
        </el-table-column>
        <el-table-column prop="lixiang" label="理想L9" align="center" />
        <el-table-column prop="xiaopeng" label="小鹏G9" align="center" />
        <el-table-column prop="wenjie" label="问界M7" align="center" />
        <el-table-column prop="tengshi" label="腾势D9" align="center" />
        <el-table-column prop="priority" label="用户关注度" align="center">
          <template #default="scope">
            <el-tag v-if="scope.row.priority === '高'" type="danger">{{ scope.row.priority }}</el-tag>
            <el-tag v-else-if="scope.row.priority === '中'" type="warning">{{
              scope.row.priority
            }}</el-tag>
            <el-tag v-else>{{ scope.row.priority }}</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 竞品满意度对比 -->
    <el-row :gutter="16" style="margin-top: 16px;">
      <el-col :span="12">
        <el-card>
          <template #header>
            <div style="font-weight: bold;">竞品满意度对比雷达图（满分5分）</div>
          </template>
          <div ref="radarChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <template #header>
            <div style="font-weight: bold;">声量份额对比图</div>
          </template>
          <div ref="shareChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 竞品情点词云对比 -->
    <el-card style="margin-top: 16px;">
      <template #header>
        <div style="font-weight: bold;">竞品情点词云对比</div>
      </template>
      <el-row :gutter="16">
        <el-col :span="8">
          <div class="word-cloud-container" style="background: #f0f9ff;">
            <h4 style="margin-bottom: 12px; text-align: center; color: #409eff;">特斯拉</h4>
            <div class="word-cloud">
              <span class="word-large">唤醒频繁</span>
              <span class="word-medium">方言无效</span>
              <span class="word-small">误触</span>
              <span class="word-large">逻辑混乱</span>
              <span class="word-small">延迟</span>
              <span class="word-medium">体验差</span>
            </div>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="word-cloud-container" style="background: #fff3e0;">
            <h4 style="margin-bottom: 12px; text-align: center; color: #e6a23c;">蔚来</h4>
            <div class="word-cloud">
              <span class="word-large">卡顿</span>
              <span class="word-medium">误唤醒</span>
              <span class="word-large">功能局限</span>
              <span class="word-small">响应慢</span>
              <span class="word-medium">不够灵敏</span>
              <span class="word-small">偶尔失败</span>
            </div>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="word-cloud-container" style="background: #fce4ec;">
            <h4 style="margin-bottom: 12px; text-align: center; color: #f56c6c;">小鹏</h4>
            <div class="word-cloud">
              <span class="word-large">耗电快</span>
              <span class="word-small">场景单一</span>
              <span class="word-large">学习成本高</span>
              <span class="word-medium">不够智能</span>
              <span class="word-small">反应慢</span>
            </div>
          </div>
        </el-col>
      </el-row>
      <el-row :gutter="16" style="margin-top: 16px;">
        <el-col :span="12">
          <div class="word-cloud-container" style="background: #e8f5e9;">
            <h4 style="margin-bottom: 12px; text-align: center; color: #67c23a;">理想</h4>
            <div class="word-cloud">
              <span class="word-large">反应慢</span>
              <span class="word-large">识别不准</span>
              <span class="word-medium">不支持连续对话</span>
              <span class="word-small">准确率低</span>
              <span class="word-medium">体验一般</span>
            </div>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="word-cloud-container" style="background: #fff9c4;">
            <h4 style="margin-bottom: 12px; text-align: center; color: #e6a23c;">比亚迪</h4>
            <div class="word-cloud">
              <span class="word-large">功能简陋</span>
              <span class="word-medium">经常失灵</span>
              <span class="word-large">交互偏硬</span>
              <span class="word-small">不够流畅</span>
              <span class="word-medium">识别差</span>
            </div>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <!-- 市场机会点分析 -->
    <el-card style="margin-top: 16px;">
      <template #header>
        <div style="font-weight: bold;">市场机会点分析</div>
      </template>
      <el-alert
        title="红海但存在显著改进机会"
        type="warning"
        :closable="false"
        style="margin-bottom: 16px;"
      >
        <p>
          蓝海 / 红海判断：竞品搭载率 45%（介于 30%-80%
          之间），我方需求声量高于行业均值
          37%，且竞品平均满意度 3.2 分（分），判定为<strong style="color: #E6A23C;"
            >红海但存在显著改进机会</strong
          >。
        </p>
      </el-alert>
      <div style="padding: 16px; background: #f5f7fa; border-radius: 8px; line-height: 2;">
        <p>
          <strong>核心机会点集中在</strong
          >：方言识别覆盖度提升、多轮对话逻辑优化、跨场景指令容兼容性増强三大方向。
        </p>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from "vue";
import * as echarts from "echarts";

const featureInput = ref("");
const radarChart = ref<HTMLElement>();
const shareChart = ref<HTMLElement>();

const comparisonData = ref([
  {
    feature: "基础唤醒",
    lantu: "支持",
    lixiang: "你好，岚图",
    xiaopeng: "你好，小P",
    wenjie: "小艺小艺",
    tengshi: "你好，腾势",
    priority: "高",
  },
  {
    feature: "唤醒成功率",
    lantu: "支持",
    lixiang: "95.2%",
    xiaopeng: "97.3%",
    wenjie: "96.8%",
    tengshi: "94.5%",
    priority: "高",
  },
  {
    feature: "平均唤应时间",
    lantu: "支持",
    lixiang: "1.1秒",
    xiaopeng: "0.8秒",
    wenjie: "0.9秒",
    tengshi: "1.3秒",
    priority: "中",
  },
  {
    feature: "四音区识别",
    lantu: "支持",
    lixiang: "支持",
    xiaopeng: "支持",
    wenjie: "支持",
    tengshi: "⚠️ 仅支音区",
    priority: "低",
  },
  {
    feature: "双音区开发",
    lantu: "不支持",
    lixiang: "支持",
    xiaopeng: "支持",
    wenjie: "⚠️ 有限支持",
    tengshi: "不支持",
    priority: "低",
  },
  {
    feature: "主驾免唤醒",
    lantu: "支持(6+播念)",
    lixiang: "支持(12+播含)",
    xiaopeng: "支持(15+播含)",
    wenjie: "支持(10+播含)",
    tengshi: "不支持",
    priority: "高",
  },
  {
    feature: "全车免唤醒",
    lantu: "不支持",
    lixiang: "支持(6+播合)",
    xiaopeng: "支持(8+播合)",
    wenjie: "不支持",
    tengshi: "不支持",
    priority: "中",
  },
  {
    feature: "连续对话",
    lantu: "支持(30秒)",
    lixiang: "支持(45秒)",
    xiaopeng: "支持(60秒)",
    wenjie: "支持(30秒)",
    tengshi: "⚠️ 支持(20秒)",
    priority: "高",
  },
]);

const analyzeCompetitor = () => {
  console.log("分析竞品:", featureInput.value);
};

const initCharts = () => {
  // 雷达图
  if (radarChart.value) {
    const chart = echarts.init(radarChart.value);
    chart.setOption({
      tooltip: {},
      legend: {
        data: ["蔚来", "小鹏", "特斯拉", "理想", "比亚迪"],
        bottom: 0,
      },
      radar: {
        indicator: [
          { name: "易用性", max: 5 },
          { name: "稳定性", max: 5 },
          { name: "创新性", max: 5 },
          { name: "性价比", max: 5 },
          { name: "美观度", max: 5 },
        ],
      },
      series: [
        {
          name: "竞品满意度",
          type: "radar",
          data: [
            { value: [4.2, 4.1, 4.3, 3.8, 4.0], name: "蔚来" },
            { value: [4.0, 3.9, 4.1, 3.7, 3.8], name: "小鹏" },
            { value: [3.8, 4.1, 3.9, 3.7, 3.7], name: "特斯拉" },
            { value: [3.5, 3.4, 3.3, 3.7, 3.5], name: "理想" },
            { value: [3.0, 3.2, 3.1, 3.8, 3.3], name: "比亚迪" },
          ],
        },
      ],
    });
  }

  // 声量份额饼图
  if (shareChart.value) {
    const chart = echarts.init(shareChart.value);
    chart.setOption({
      tooltip: {
        trigger: "item",
        formatter: "{b}: {c}% ({d}%)",
      },
      legend: {
        orient: "vertical",
        left: "left",
      },
      series: [
        {
          name: "声量份额",
          type: "pie",
          radius: "60%",
          data: [
            { value: 28, name: "我方需求声量", itemStyle: { color: "#5470C6" } },
            { value: 22, name: "特斯拉", itemStyle: { color: "#91CC75" } },
            { value: 19, name: "蔚来", itemStyle: { color: "#FAC858" } },
            { value: 17, name: "小鹏", itemStyle: { color: "#EE6666" } },
            { value: 10, name: "理想", itemStyle: { color: "#73C0DE" } },
            { value: 4, name: "比亚迪", itemStyle: { color: "#3BA272" } },
          ],
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: "rgba(0, 0, 0, 0.5)",
            },
          },
          label: {
            formatter: "{b}\n{c}%",
          },
        },
      ],
    });
  }
};

onMounted(() => {
  nextTick(() => {
    initCharts();
  });
});
</script>

<style scoped>
.competitor-page {
  padding: 20px;
  min-height: 100%;
}

.header-card h2 {
  margin: 0;
  color: #303133;
}

.word-cloud-container {
  padding: 20px;
  border-radius: 8px;
  min-height: 200px;
}

.word-cloud {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
  justify-content: center;
  padding: 20px 10px;
}

.word-cloud span {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.6);
  transition: all 0.3s;
  cursor: default;
  font-weight: 500;
}

.word-cloud span:hover {
  transform: scale(1.1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.word-large {
  font-size: 24px !important;
  font-weight: 600 !important;
  color: #303133;
}

.word-medium {
  font-size: 18px !important;
  color: #606266;
}

.word-small {
  font-size: 14px !important;
  color: #909399;
}
</style>
