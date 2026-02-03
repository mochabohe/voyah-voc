<template>
  <div class="voice-detection-page">
    <el-card class="header-card">
      <h2>声量智能探测 - {{ currentRequirement }}</h2>
      <p style="margin-top: 12px; color: #606266; font-size: 14px;">
        基于VOC大数据的智能分析结果
      </p>
    </el-card>

    <!-- 分析结果 -->
    <div>
      <!-- 声量趋势和情感分布 -->
      <el-row :gutter="16" style="margin-top: 16px;">
        <el-col :span="12">
          <el-card>
            <template #header>
              <div style="font-weight: bold;">声量趋势（近90天）</div>
            </template>
            <div ref="trendChart" style="height: 320px;"></div>
          </el-card>
        </el-col>
        <el-col :span="12">
          <el-card>
            <template #header>
              <div style="font-weight: bold;">情感分布</div>
            </template>
            <div ref="emotionChart" style="height: 320px;"></div>
          </el-card>
        </el-col>
      </el-row>

      <!-- 用户画像 -->
      <el-card style="margin-top: 16px;">
        <template #header>
          <div style="font-weight: bold;">用户画像</div>
        </template>
        <el-row :gutter="16">
          <el-col :span="6">
            <div ref="ageChart" style="height: 250px;"></div>
          </el-col>
          <el-col :span="6">
            <div ref="incomeChart" style="height: 250px;"></div>
          </el-col>
          <el-col :span="6">
            <div ref="regionChart" style="height: 250px;"></div>
          </el-col>
          <el-col :span="6">
            <div ref="carModelChart" style="height: 250px;"></div>
          </el-col>
        </el-row>
      </el-card>

      <!-- 典型用户原声列表 -->
      <el-card style="margin-top: 16px;">
        <template #header>
          <div style="font-weight: bold;">典型用户原声列表（展示最具代表性的用户原始反馈）</div>
        </template>
        
        <el-tabs v-model="activeTab" type="card">
          <!-- 全部 -->
          <el-tab-pane label="全部" name="all">
            <el-table :data="allFeedbackList" style="width: 100%">
              <el-table-column type="index" label="#" width="60" />
              <el-table-column prop="user" label="用户" width="120">
                <template #default="scope">
                  <el-avatar :size="32">{{ scope.row.user }}</el-avatar>
                </template>
              </el-table-column>
              <el-table-column prop="content" label="反馈内容" />
              <el-table-column prop="source" label="来源" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.sourceType" size="small">{{ scope.row.source }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="emotion" label="情感" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.emotionType">{{ scope.row.emotion }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="score" label="点赞数" width="100" />
            </el-table>
          </el-tab-pane>

          <!-- 公域 -->
          <el-tab-pane label="公域" name="public">
            <el-table :data="publicFeedbackList" style="width: 100%">
              <el-table-column type="index" label="#" width="60" />
              <el-table-column prop="user" label="用户" width="120">
                <template #default="scope">
                  <el-avatar :size="32">{{ scope.row.user }}</el-avatar>
                </template>
              </el-table-column>
              <el-table-column prop="content" label="反馈内容" />
              <el-table-column prop="platform" label="平台" width="120">
                <template #default="scope">
                  <el-tag type="info" size="small">{{ scope.row.platform }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="emotion" label="情感" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.emotionType">{{ scope.row.emotion }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="score" label="点赞数" width="100" />
            </el-table>
          </el-tab-pane>

          <!-- 私域 -->
          <el-tab-pane label="私域" name="private">
            <el-table :data="privateFeedbackList" style="width: 100%">
              <el-table-column type="index" label="#" width="60" />
              <el-table-column prop="user" label="用户" width="120">
                <template #default="scope">
                  <el-avatar :size="32">{{ scope.row.user }}</el-avatar>
                </template>
              </el-table-column>
              <el-table-column prop="content" label="反馈内容" />
              <el-table-column prop="channel" label="渠道" width="120">
                <template #default="scope">
                  <el-tag type="success" size="small">{{ scope.row.channel }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="emotion" label="情感" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.emotionType">{{ scope.row.emotion }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="dealStatus" label="处理状态" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.dealType" size="small">{{ scope.row.dealStatus }}</el-tag>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </el-card>

      <!-- 数据简报 -->
      <el-card style="margin-top: 16px;">
        <template #header>
          <div style="font-weight: bold;">数据简报（自动生成一段文字摘要）</div>
        </template>
        <el-alert type="info" :closable="false">
          <div style="line-height: 2;">
            近90天，关于"<strong>{{ currentRequirement }}</strong>"的讨论共 <strong style="color: #409EFF;">1,234条</strong>，其中 <strong style="color: #67C23A;">78%</strong> 为期待性正面表达，<strong style="color: #F56C6C;">22%</strong> 为愤怒性负面表达。
            主要提及用户为<strong>北方地区（65%）的ES6/EC6车主</strong>。
            <br/>
            <strong>核心诉求：</strong>用户普遍期待该功能能够在冬季使用时提升舒适性，但部分用户反馈现有配置车型未搭载此功能，表达不满。
          </div>
        </el-alert>
      </el-card>

  
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import * as echarts from "echarts";

const activeTab = ref("all");

const trendChart = ref<HTMLElement>();
const emotionChart = ref<HTMLElement>();
const ageChart = ref<HTMLElement>();
const incomeChart = ref<HTMLElement>();
const regionChart = ref<HTMLElement>();
const carModelChart = ref<HTMLElement>();

// 全部反馈列表
const allFeedbackList = ref([
  { user: "张", content: "希望能像小鹏那样连续对话，不用一直喊'你好岚图'，开车时真的很方便", source: "公域", sourceType: "info", emotion: "期待", emotionType: "success", score: 156 },
  { user: "李", content: "广东人表示很需要粤语识别，现在普通话说得我自己都尴尬", source: "公域", sourceType: "info", emotion: "期待", emotionType: "success", score: 98 },
  { user: "王", content: "现在的语音助手就是个摆设，十次有三次听不懂，还不如手动按", source: "私域", sourceType: "success", emotion: "愤怒", emotionType: "danger", score: 234 },
  { user: "赵", content: "建议增加四川话识别，我每次说话它都理解错", source: "公域", sourceType: "info", emotion: "建议", emotionType: "warning", score: 67 },
  { user: "周", content: "APP反馈了好多次都没有回复，这个功能什么时候能上线？", source: "私域", sourceType: "success", emotion: "不满", emotionType: "danger", score: 145 },
]);

// 公域反馈列表
const publicFeedbackList = ref([
  { user: "张", content: "希望能像小鹏那样连续对话，不用一直喊'你好岚图'，开车时真的很方便", platform: "懂车帝", emotion: "期待", emotionType: "success", score: 156 },
  { user: "李", content: "广东人表示很需要粤语识别，现在普通话说得我自己都尴尬", platform: "汽车之家", emotion: "期待", emotionType: "success", score: 98 },
  { user: "赵", content: "建议增加四川话识别，我每次说话它都理解错", platform: "小红书", emotion: "建议", emotionType: "warning", score: 67 },
  { user: "孙", content: "看到蔚来都有了，什么时候岚图能跟上？", platform: "抖音", emotion: "期待", emotionType: "success", score: 89 },
]);

// 私域反馈列表
const privateFeedbackList = ref([
  { user: "王", content: "现在的语音助手就是个摆设，十次有三次听不懂，还不如手动按", channel: "官方APP", emotion: "愤怒", emotionType: "danger", dealStatus: "已受理", dealType: "warning" },
  { user: "周", content: "APP反馈了好多次都没有回复，这个功能什么时候能上线？", channel: "客服系统", emotion: "不满", emotionType: "danger", dealStatus: "处理中", dealType: "warning" },
  { user: "吴", content: "希望能像小鹏一样支持连续对话，每次都要唤醒太麻烦", channel: "官方APP", emotion: "建议", emotionType: "warning", dealStatus: "已完成", dealType: "success" },
  { user: "郑", content: "语音识别准确率太低了，尤其是在高速行驶时", channel: "400电话", emotion: "不满", emotionType: "danger", dealStatus: "已受理", dealType: "warning" },
]);

// 当前需求名称（从localStorage读取）
const currentRequirement = ref(localStorage.getItem('selectedRequirement') || '方向盘加热');

const initCharts = () => {
  // 声量趋势图
  if (trendChart.value) {
    const chart = echarts.init(trendChart.value);
    chart.setOption({
      tooltip: { trigger: "axis" },
     xAxis: {
        type: "category",
        data: ["第1周", "第2周", "第3周", "第4周", "第5周", "第6周", "第7周", "第8周", "第9周", "第10周", "第11周", "第12周"],
      },
      yAxis: { type: "value", name: "讨论量" },
      series: [
        {
          data: [45, 62, 78, 95, 112, 135, 158, 182, 205, 228, 245, 260],
          type: "line",
          smooth: true,
          areaStyle: { color: "rgba(64, 158, 255, 0.1)" },
          itemStyle: { color: "#409EFF" },
        },
      ],
    });
  }

  // 情感分布饼图
  if (emotionChart.value) {
    const chart = echarts.init(emotionChart.value);
    chart.setOption({
      tooltip: { trigger: "item" },
      legend: { orient: "vertical", left: "left" },
      series: [
        {
          name: "情感",
          type: "pie",
          radius: "60%",
          data: [
            { value: 78, name: "正面期待", itemStyle: { color: "#67C23A" } },
            { value: 22, name: "负面吐槽", itemStyle: { color: "#F56C6C" } },
          ],
          label: { formatter: "{b}\n{d}%" },
        },
      ],
    });
  }

  // 年龄分布
  if (ageChart.value) {
    const chart = echarts.init(ageChart.value);
    chart.setOption({
      title: { text: "年龄分布", left: "center" },
      tooltip: {},
      xAxis: { type: "category", data: ["25-30", "31-35", "36-40", "41-45", "46+"] },
      yAxis: { type: "value" },
      series: [{ data: [12, 28, 35, 20, 5], type: "bar", itemStyle: { color: "#5470C6" } }],
    });
  }

  // 收入分布
  if (incomeChart.value) {
    const chart = echarts.init(incomeChart.value);
    chart.setOption({
      title: { text: "家庭年收入", left: "center" },
      tooltip: {},
      xAxis: { type: "category", data: ["20-30万", "30-50万", "50-80万", "80万+"] },
      yAxis: { type: "value" },
      series: [{ data: [15, 45, 30, 10], type: "bar", itemStyle: { color: "#91CC75" } }],
    });
  }

  // 地区分布
  if (regionChart.value) {
    const chart = echarts.init(regionChart.value);
    chart.setOption({
      title: { text: "地区分布", left: "center" },
      tooltip: { trigger: "item" },
      series: [
        {
          type: "pie",
          radius: "50%",
          data: [
            { value: 65, name: "北方", itemStyle: { color: "#5470C6" } },
            { value: 35, name: "南方", itemStyle: { color: "#91CC75" } },
          ],
        },
      ],
    });
  }

  // 车型分布
  if (carModelChart.value) {
    const chart = echarts.init(carModelChart.value);
    chart.setOption({
      title: { text: "车型分布", left: "center" },
      tooltip: { trigger: "item" },
      series: [
        {
          type: "pie",
          radius: "50%",
          data: [
            { value: 45, name: "ES6", itemStyle: { color: "#5470C6" } },
            { value: 35, name: "EC6", itemStyle: { color: "#91CC75" } },
            { value: 20, name: "其他", itemStyle: { color: "#FAC858" } },
          ],
        },
      ],
    });
  }
};

// 页面加载时初始化图表
onMounted(() => {
  initCharts();
});
</script>

<style scoped>
.voice-detection-page {
  padding: 20px;
  min-height: 100%;
}

.header-card h2 {
  margin: 0;
  color: #303133;
}

.qa-example {
  padding: 20px;
}

.question,
.answer {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
}

.question .content,
.answer .content {
  flex: 1;
}

.label {
  font-weight: bold;
  color: #303133;
  margin-bottom: 8px;
}

.text {
  color: #606266;
  line-height: 1.8;
}

.answer {
  padding-left: 44px;
}
</style>
