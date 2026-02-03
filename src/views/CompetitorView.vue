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
          <el-button type="primary" @click="analyzeCompetitor" :loading="loading">
            <el-icon v-if="!loading"><Search /></el-icon>
            对比分析
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <div v-if="showResults">
    <!-- 竞品功能对比矩阵 -->
    <el-card style="margin-top: 16px;">
      <template #header>
        <div style="font-weight: bold;">竞品功能对标矩阵</div>
      </template>
      <el-table :data="comparisonData" border style="width: 100%">
        <el-table-column prop="feature" label="功能点" width="120" fixed />
        
        <!-- 通用的单元格渲染模板插槽 -->
        <el-table-column prop="lantu" label="岚图梦想家" min-width="140">
          <template #default="scope">
            <div class="cell-content">
              <el-icon v-if="scope.row.lantu.includes('不')" color="#F56C6C"><CloseBold /></el-icon>
              <el-icon v-else-if="scope.row.lantu.includes('仅') || scope.row.lantu.includes('有限')" color="#E6A23C"><Warning /></el-icon>
              <!-- 只有'基础唤醒'或'四音区'等明确支持/有内容的才显示勾选，百分比和秒数不显示勾选 -->
              <el-icon v-else-if="!scope.row.lantu.includes('%') && !scope.row.lantu.includes('秒') && scope.row.lantu !== '全部达标'" color="#67C23A"><Select /></el-icon>
              <span>{{ scope.row.lantu }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="lixiang" label="理想 L9" min-width="140">
          <template #default="scope">
            <div class="cell-content">
              <el-icon v-if="scope.row.lixiang.includes('不')" color="#F56C6C"><CloseBold /></el-icon>
              <el-icon v-else-if="scope.row.lixiang.includes('仅') || scope.row.lixiang.includes('有限')" color="#E6A23C"><Warning /></el-icon>
              <el-icon v-else-if="!scope.row.lixiang.includes('%') && !scope.row.lixiang.includes('秒')" color="#67C23A"><Select /></el-icon>
              <span>{{ scope.row.lixiang }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="xiaopeng" label="小鹏 G9" min-width="140">
          <template #default="scope">
            <div class="cell-content">
              <el-icon v-if="scope.row.xiaopeng.includes('不')" color="#F56C6C"><CloseBold /></el-icon>
              <el-icon v-else-if="scope.row.xiaopeng.includes('仅') || scope.row.xiaopeng.includes('有限')" color="#E6A23C"><Warning /></el-icon>
              <el-icon v-else-if="!scope.row.xiaopeng.includes('%') && !scope.row.xiaopeng.includes('秒')" color="#67C23A"><Select /></el-icon>
              <span>{{ scope.row.xiaopeng }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="wenjie" label="问界 M7" min-width="140">
          <template #default="scope">
            <div class="cell-content">
              <el-icon v-if="scope.row.wenjie.includes('不')" color="#F56C6C"><CloseBold /></el-icon>
              <el-icon v-else-if="scope.row.wenjie.includes('仅') || scope.row.wenjie.includes('有限')" color="#E6A23C"><Warning /></el-icon>
              <el-icon v-else-if="!scope.row.wenjie.includes('%') && !scope.row.wenjie.includes('秒')" color="#67C23A"><Select /></el-icon>
              <span>{{ scope.row.wenjie }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="tengshi" label="腾势 D9" min-width="140">
          <template #default="scope">
            <div class="cell-content">
              <el-icon v-if="scope.row.tengshi.includes('不')" color="#F56C6C"><CloseBold /></el-icon>
              <el-icon v-else-if="scope.row.tengshi.includes('仅') || scope.row.tengshi.includes('有限')" color="#E6A23C"><Warning /></el-icon>
              <el-icon v-else-if="!scope.row.tengshi.includes('%') && !scope.row.tengshi.includes('秒')" color="#67C23A"><Select /></el-icon>
              <span>{{ scope.row.tengshi }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="best" label="行业最佳" min-width="120" />

        <el-table-column prop="priority" label="用户关注度" width="100" align="center">
          <template #default="scope">
            <span :style="{ color: scope.row.priority === '高' ? '#F56C6C' : scope.row.priority === '中' ? '#E6A23C' : '#909399', fontWeight: 'bold' }">
              {{ scope.row.priority }}
            </span>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 竞品满意度对比 -->
    <el-row :gutter="16" style="margin-top: 16px;">
      <el-col :span="12">
        <el-card>
          <template #header>
            <div style="font-weight: bold;">竞品满意度</div>
          </template>
          <div ref="radarChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <template #header>
            <div style="font-weight: bold;">声量份额</div>
          </template>
          <div ref="shareChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 竞品痛点词云对比 -->
    <el-card style="margin-top: 16px;">
      <template #header>
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <div style="font-weight: bold;">竞品槽点</div>
          <el-radio-group v-model="viewMode" size="small">
            <el-radio-button value="bar">柱状图</el-radio-button>
            <el-radio-button value="cloud">词云</el-radio-button>
          </el-radio-group>
        </div>
      </template>

      <!-- 柱状图视图 -->
      <div v-show="viewMode === 'bar'">
        <el-row :gutter="16">
          <el-col :span="8">
            <div ref="teslaBarChart" style="height: 300px;"></div>
          </el-col>
          <el-col :span="8">
            <div ref="nioBarChart" style="height: 300px;"></div>
          </el-col>
          <el-col :span="8">
            <div ref="xiaopengBarChart" style="height: 300px;"></div>
          </el-col>
        </el-row>
        <el-row :gutter="16" style="margin-top: 16px;">
          <el-col :span="12">
            <div ref="lixiangBarChart" style="height: 300px;"></div>
          </el-col>
          <el-col :span="12">
            <div ref="bydBarChart" style="height: 300px;"></div>
          </el-col>
        </el-row>
      </div>

      <!-- 词云视图 -->
      <div v-show="viewMode === 'cloud'">
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
      </div>
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
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, watch, inject, type Ref } from "vue";
import * as echarts from "echarts";
import type { ECharts } from "echarts";
import { Search, CloseBold, Warning, Select } from "@element-plus/icons-vue";

const featureInput = ref("");
const radarChart = ref<HTMLElement>();
const shareChart = ref<HTMLElement>();
const showResults = ref(false);
const loading = ref(false);

// 5个品牌的柱状图refs
const teslaBarChart = ref<HTMLElement>();
const nioBarChart = ref<HTMLElement>();
const xiaopengBarChart = ref<HTMLElement>();
const lixiangBarChart = ref<HTMLElement>();
const bydBarChart = ref<HTMLElement>();

const viewMode = ref<"bar" | "cloud">("bar"); // 默认显示柱状图

let chartInstances: ECharts[] = [];

// 竞品痛点数据（包含权重，对应词云中的字体大小）
const painPointsData = [
  { 
    brand: "特斯拉", 
    color: "#409EFF",
    bgColor: "#f0f9ff",
    points: [
      { name: "唤醒频繁", weight: 100 },  // large
      { name: "逻辑混乱", weight: 100 },  // large
      { name: "方言无效", weight: 70 },   // medium
      { name: "体验差", weight: 70 },     // medium
      { name: "误触", weight: 40 },       // small
      { name: "延迟", weight: 40 }        // small
    ]
  },
  { 
    brand: "蔚来", 
    color: "#E6A23C",
    bgColor: "#fff3e0",
    points: [
      { name: "卡顿", weight: 100 },      // large
      { name: "功能局限", weight: 100 },  // large
      { name: "误唤醒", weight: 70 },     // medium
      { name: "不够灵敏", weight: 70 },   // medium
      { name: "响应慢", weight: 40 },     // small
      { name: "偶尔失败", weight: 40 }    // small
    ]
  },
  { 
    brand: "小鹏", 
    color: "#F56C6C",
    bgColor: "#fce4ec",
    points: [
      { name: "耗电快", weight: 100 },    // large
      { name: "学习成本高", weight: 100 }, // large
      { name: "不够智能", weight: 70 },   // medium
      { name: "场景单一", weight: 40 },   // small
      { name: "反应慢", weight: 40 }      // small
    ]
  },
  { 
    brand: "理想", 
    color: "#67C23A",
    bgColor: "#e8f5e9",
    points: [
      { name: "反应慢", weight: 100 },    // large
      { name: "识别不准", weight: 100 },  // large
      { name: "不支持连续对话", weight: 70 }, // medium
      { name: "体验一般", weight: 70 },   // medium
      { name: "准确率低", weight: 40 }    // small
    ]
  },
  { 
    brand: "比亚迪", 
    color: "#FAC858",
    bgColor: "#fff9c4",
    points: [
      { name: "功能简陋", weight: 100 },  // large
      { name: "交互偏硬", weight: 100 },  // large
      { name: "经常失灵", weight: 70 },   // medium
      { name: "识别差", weight: 70 },     // medium
      { name: "不够流畅", weight: 40 }    // small
    ]
  }
];

const comparisonData = ref([
  {
    feature: "基础唤醒",
    lantu: "你好, 岚图",
    lixiang: "理想同学",
    xiaopeng: "你好, 小P",
    wenjie: "小艺小艺",
    tengshi: "你好, 腾势",
    best: "全部达标",
    priority: "高",
  },
  {
    feature: "唤醒成功率",
    lantu: "95.2%",
    lixiang: "97.3%",
    xiaopeng: "96.8%",
    wenjie: "96.1%",
    tengshi: "94.5%",
    best: "理想 L9",
    priority: "高",
  },
  {
    feature: "平均响应时间",
    lantu: "1.1秒",
    lixiang: "0.8秒",
    xiaopeng: "0.6秒",
    wenjie: "0.9秒",
    tengshi: "1.3秒",
    best: "小鹏 G9",
    priority: "中",
  },
  {
    feature: "四音区识别",
    lantu: "支持",
    lixiang: "支持",
    xiaopeng: "支持",
    wenjie: "支持",
    tengshi: "仅三音区", // 修正：仅支音区
    best: "岚图/理想/小鹏/问界",
    priority: "低",
  },
  {
    feature: "双音区并发", // 修正：双音区开发
    lantu: "不支持",
    lixiang: "支持",
    xiaopeng: "支持",
    wenjie: "有限支持",
    tengshi: "不支持",
    best: "理想/小鹏",
    priority: "低",
  },
  {
    feature: "主驾免唤醒",
    lantu: "支持(8个指令)",
    lixiang: "支持(12个指令)",
    xiaopeng: "支持(15个指令)",
    wenjie: "支持(10个指令)",
    tengshi: "不支持",
    best: "小鹏 G9",
    priority: "高",
  },
  {
    feature: "全车免唤醒",
    lantu: "不支持",
    lixiang: "支持(6个指令)",
    xiaopeng: "支持(8个指令)",
    wenjie: "不支持",
    tengshi: "不支持",
    best: "小鹏 G9",
    priority: "中",
  },
  {
    feature: "连续对话",
    lantu: "支持(30秒)",
    lixiang: "支持(45秒)",
    xiaopeng: "支持(60秒)",
    wenjie: "支持(30秒)",
    tengshi: "支持(20秒)",
    best: "小鹏 G9",
    priority: "高",
  },
]);

const analyzeCompetitor = () => {
  if (!featureInput.value) {
    // 允许空输入，或者可以添加提示
  }
  
  loading.value = true;
  console.log("分析竞品:", featureInput.value);
  
  // 模拟分析过程
  setTimeout(() => {
    loading.value = false;
    showResults.value = true;
    
    // M9 OTA 专题数据覆盖
    if (featureInput.value && featureInput.value.includes('OTA')) {
      comparisonData.value = [
        { feature: "智能驾驶版本", lantu: "V1.3.0", lixiang: "OTA 5.0", xiaopeng: "XOS 4.4", wenjie: "HarmonyOS 4", tengshi: "V1.2", best: "HarmonyOS 4", priority: "高" },
        { feature: "城市NCA覆盖", lantu: "部分开通", lixiang: "全量推送", xiaopeng: "243城", wenjie: "全国都能开", tengshi: "测试中", best: "全国都能开", priority: "高" },
        { feature: "智慧大灯交互", lantu: "LED矩阵", lixiang: "LED", xiaopeng: "LED", wenjie: "百万像素光场屏", tengshi: "LED", best: "百万像素光场屏", priority: "中" },
        { feature: "底盘技术", lantu: "CDC+空悬", lixiang: "魔毯空悬", xiaopeng: "双腔空悬", wenjie: "途灵智能底盘", tengshi: "云辇-C", best: "途灵智能底盘", priority: "High" },
        { feature: "泊车能力", lantu: "APA自动泊车", lixiang: "代客泊车", xiaopeng: "跨楼层记忆", wenjie: "代客泊车+机械车位", tengshi: "自动泊车", best: "代客泊车+机械车位", priority: "中" }
      ];
    }

    // 显示结果后，DOM更新了再初始化图表
    nextTick(() => {
      initCharts();
    });
  }, 800);
};

// 创建单个品牌的柱状图
const createBrandBarChart = (chartElement: HTMLElement | undefined, brandData: typeof painPointsData[0]) => {
  if (!chartElement) return null;
  
  const chartInstance = echarts.init(chartElement);
  
  chartInstance.setOption({
    title: {
      text: brandData.brand,
      left: 'center',
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold',
        color: brandData.color
      }
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      },
      formatter: (params: any) => {
        const data = params[0];
        return `<strong>${data.name}</strong><br/>权重: ${data.value}`;
      }
    },
    grid: {
      left: '5%',
      right: '5%',
      bottom: '15%',
      top: '60px',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: brandData.points.map(p => p.name),
      axisLabel: {
        interval: 0,
        rotate: 45,
        fontSize: 11,
        color: '#606266'
      }
    },
    yAxis: {
      type: 'value',
      name: '严重程度',
      max: 120,
      axisLabel: {
        fontSize: 11
      }
    },
    series: [
      {
        name: '痛点权重',
        type: 'bar',
        data: brandData.points.map(p => ({
          value: p.weight,
          itemStyle: {
            color: brandData.color
          }
        })),
        barWidth: '60%',
        label: {
          show: true,
          position: 'top',
          fontSize: 11,
          formatter: (params: any) => {
            if (params.value >= 100) return '高';
            if (params.value >= 70) return '中';
            return '低';
          }
        }
      }
    ]
  });
  
  return chartInstance;
};

const initBrandBarCharts = () => {
  // 清除旧实例
  chartInstances.forEach(instance => instance?.dispose());
  chartInstances = [];
  
  // 创建5个品牌的柱状图
  const chart1 = createBrandBarChart(teslaBarChart.value, painPointsData[0]);
  const chart2 = createBrandBarChart(nioBarChart.value, painPointsData[1]);
  const chart3 = createBrandBarChart(xiaopengBarChart.value, painPointsData[2]);
  const chart4 = createBrandBarChart(lixiangBarChart.value, painPointsData[3]);
  const chart5 = createBrandBarChart(bydBarChart.value, painPointsData[4]);
  
  if (chart1) chartInstances.push(chart1);
  if (chart2) chartInstances.push(chart2);
  if (chart3) chartInstances.push(chart3);
  if (chart4) chartInstances.push(chart4);
  if (chart5) chartInstances.push(chart5);
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
            { value: 28, name: "岚图", itemStyle: { color: "#5470C6" } },
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

  // 品牌柱状图
  initBrandBarCharts();
};

// 监听视图模式变化,在切换到柱状图时重新渲染
watch(viewMode, (newMode) => {
  if (newMode === 'bar' && showResults.value) { // 只有结果显示了才处理
    nextTick(() => {
      initBrandBarCharts();
    });
  }
});

// 窗口大小变化时重绘图表
window.addEventListener('resize', () => {
  chartInstances.forEach(instance => instance?.resize());
});

// 获取导航参数
const navigationParams = inject('navigationParams') as Ref<any>;

// 监听参数变化
watch(() => navigationParams?.value, (newVal) => {
  if (newVal?.topic) {
    featureInput.value = newVal.topic;
    analyzeCompetitor();
  }
}, { immediate: true });

onMounted(() => {
  // onMounted 逻辑已由 immediate watcher 覆盖，或者可以在此做其他初始化
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

.cell-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}
</style>
