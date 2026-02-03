<template>
  <div class="monitoring-page">
    <!-- 统计卡片 -->
    <el-row :gutter="20" style="margin-bottom: 20px">
      <el-col :span="8">
        <div class="stat-card running">
          <div class="stat-number">12</div>
          <div class="stat-label">运行中</div>
        </div>
      </el-col>
      <el-col :span="8">
        <div class="stat-card triggered">
          <div class="stat-number">3</div>
          <div class="stat-label">已触发</div>
        </div>
      </el-col>
      <el-col :span="8">
        <div class="stat-card disabled">
          <div class="stat-number">2</div>
          <div class="stat-label">已禁用</div>
        </div>
      </el-col>
    </el-row>

    <!-- 最近触发预警 -->
    <el-card shadow="hover" style="margin-bottom: 20px">
      <template #header>
        <div class="card-header">
          <div>
            <el-icon><Warning /></el-icon>
            <span>最近触发预警</span>
          </div>
        </div>
      </template>
      <el-timeline>
        <el-timeline-item color="#F56C6C" size="large">
          <el-card>
            <div class="alert-item">
              <div class="alert-content">
                <h4>
                  <el-tag type="danger" size="small">紧急</el-tag>
                  H56D充电慢增长超20%
                </h4>
                <p style="margin: 8px 0; color: #606266">触发时间：3分钟前</p>
              </div>
              <div class="alert-actions">
                <el-button type="primary" size="small">查看详情</el-button>
                <el-button size="small">标记已处理</el-button>
              </div>
            </div>
          </el-card>
        </el-timeline-item>
        <el-timeline-item color="#E6A23C" size="large">
          <el-card>
            <div class="alert-item">
              <div class="alert-content">
                <h4>
                  <el-tag type="warning" size="small">警告</el-tag>
                  H77车机卡顿数量上升超10%
                </h4>
                <p style="margin: 8px 0; color: #606266">触发时间：1小时前</p>
              </div>
              <div class="alert-actions">
                <el-button type="primary" size="small">查看详情</el-button>
                <el-button size="small">标记已处理</el-button>
              </div>
            </div>
          </el-card>
        </el-timeline-item>
      </el-timeline>
    </el-card>

    <!-- 监控任务列表 -->
    <el-card shadow="hover" style="margin-bottom: 20px">
      <template #header>
        <div class="card-header">
          <div>
            <el-icon><List /></el-icon>
            <span>监控任务列表</span>
          </div>
          <div style="display: flex; gap: 10px;">
            <el-button
              size="small"
              @click="showRecommendDialog = true"
            >
              <el-icon><MagicStick /></el-icon>
              智能推荐
            </el-button>
            <el-button
              type="primary"
              size="small"
              @click="showCreateDialog = true"
            >
              <el-icon><Plus /></el-icon>
              创建监控规则
            </el-button>
          </div>
        </div>
      </template>

      <div class="task-list">
        <div 
          v-for="(task, index) in monitoringTasks" 
          :key="index" 
          class="task-item"
        >
          <div class="task-status">
            <el-tag 
              :type="task.status === 'running' ? 'success' : 'info'" 
              size="small"
            >
              {{ task.status === 'running' ? '● 运行中' : '⏸ 已暂停' }}
            </el-tag>
          </div>
          <div class="task-info">
            <h4>{{ task.name }}</h4>
            <div class="task-meta">
              <span>频率: {{ task.frequency }}</span>
              <span>上次检查: {{ task.lastCheck }}</span>
              <span>触发次数: {{ task.triggerCount }}</span>
            </div>
          </div>
          <div class="task-actions">
            <el-button-group>
              <el-button size="small" :icon="Edit" @click="editTask(task.name)" />
              <el-button size="small" :icon="VideoPause" @click="pauseTask(task.name)" />
              <el-button size="small" :icon="Delete" @click="deleteTask(task.name)" />
            </el-button-group>
          </div>
        </div>
      </div>
    </el-card>

    <!-- 创建监控规则对话框 -->
    <el-dialog v-model="showCreateDialog" title="创建监控规则" width="700px">
      <el-tabs v-model="createMethod">
        <el-tab-pane label="自然语言创建" name="natural">
          <div class="natural-input">
            <el-input
              v-model="naturalLanguage"
              type="textarea"
              :rows="4"
              placeholder="例如：当H56D充电慢的负面反馈在3天内增加超过20%时通知我"
            />
            <el-button
              type="primary"
              style="margin-top: 15px"
              @click="parseNaturalLanguage"
            >
              <el-icon><MagicStick /></el-icon>
              AI解析
            </el-button>
          </div>
        </el-tab-pane>

        <el-tab-pane label="表单配置" name="form">
          <el-form :model="ruleForm" label-width="100px">
            <el-form-item label="监控对象">
              <el-cascader
                v-model="ruleForm.target"
                :options="carModelOptions"
                placeholder="请选择车型"
                style="width: 100%"
              />
              <div style="font-size: 12px; color: #909399; margin-top: 5px;">
                支持多层级筛选：FREE/追光/知音/梦想家/秦山
              </div>
            </el-form-item>

            <el-form-item label="监控指标">
              <el-select v-model="ruleForm.metric" placeholder="请选择指标">
                <el-option label="负面反馈数量" value="negative_count" />
                <el-option label="正面反馈数量" value="positive_count" />
              </el-select>
            </el-form-item>

            <el-form-item label="时间窗口">
              <el-select v-model="ruleForm.timeWindow" style="width: 100%">
                <el-option label="1天" value="1d" />
                <el-option label="3天" value="3d" />
                <el-option label="7天" value="7d" />
                <el-option label="30天" value="30d" />
              </el-select>
            </el-form-item>

            <el-form-item label="阈值条件">
              <div style="display: flex; align-items: center; gap: 8px;">
                <el-select v-model="ruleForm.condition" style="width: 120px;">
                  <el-option label="增长超过" value="increase" />
                  <el-option label="减少超过" value="decrease" />
                </el-select>
                <el-input
                  v-model="ruleForm.threshold"
                  style="width: 100px;"
                  placeholder="20"
                />
                <span>%</span>
              </div>
            </el-form-item>
          </el-form>
        </el-tab-pane>
      </el-tabs>

      <template #footer>
        <el-button @click="showCreateDialog = false">取消</el-button>
        <el-button type="primary" @click="createRule">
          <el-icon><Check /></el-icon>
          保存规则
        </el-button>
      </template>
    </el-dialog>

    <!-- 智能推荐对话框 -->
    <el-dialog v-model="showRecommendDialog" title="智能推荐监控规则" width="800px">
      <div style="margin-bottom: 20px; padding: 15px; background: #f0f9ff; border-radius: 8px; border-left: 4px solid #409EFF;">
        <div style="display: flex; align-items: center; gap: 10px; margin-bottom: 10px;">
          <el-icon color="#409EFF" :size="20"><MagicStick /></el-icon>
          <span style="font-weight: 600; color: #409EFF;">AI智能分析中...</span>
        </div>
        <p style="margin: 0; font-size: 14px; color: #606266;">基于近30天VOC数据，为您推荐以下高价值监控规则</p>
      </div>

      <div class="recommend-list">
        <div 
          v-for="(rec, index) in recommendedRules" 
          :key="index" 
          class="recommend-item"
          :class="{ 'selected': selectedRecommendation === rec }"
          @click="selectRecommendation(rec)"
        >
          <div class="recommend-header">
            <el-tag :type="rec.priority === '高' ? 'danger' : rec.priority === '中' ? 'warning' : 'info'" size="small">
              {{ rec.priority }}优先级
            </el-tag>
            <span class="recommend-title">{{ rec.title }}</span>
          </div>
          <div class="recommend-desc">{{ rec.description }}</div>
          <div class="recommend-stats">
            <span>📊 近期声量: <strong>{{ rec.volume }}</strong></span>
            <span>📈 增长趋势: <strong style="color: #F56C6C;">{{ rec.trend }}</strong></span>
            <span>⚠️ 风险等级: <strong>{{ rec.risk }}</strong></span>
          </div>
        </div>
      </div>

      <template #footer>
        <el-button @click="showRecommendDialog = false">关闭</el-button>
        <el-button type="primary" @click="createFromRecommendation">应用选中规则</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import {
  Edit,
  VideoPause,
  Delete,
  Plus,
  MagicStick,
  Check,
} from "@element-plus/icons-vue";

const showCreateDialog = ref(false);
const showRecommendDialog = ref(false);
const createMethod = ref("natural");

// 监控任务列表
const monitoringTasks = ref([
  {
    name: "H56D充电问题监控",
    status: "running",
    frequency: "每小时",
    lastCheck: "10分钟前",
    triggerCount: 3
  },
  {
    name: "A7车机系统监控",
    status: "running",
    frequency: "每小时",
    lastCheck: "15分钟前",
    triggerCount: 5
  }
]);
const naturalLanguage = ref("");
const selectedRecommendation = ref<any>(null);

// 智能推荐的监控规则列表
const recommendedRules = ref([
  {
    priority: '高',
    title: 'H56D充电慢问题监控',
    description: '近7天负面反馈增长42%，建议立即监控',
    volume: '156次',
    trend: '+42%',
    risk: '高',
    config: {
      target: ['梦想家', 'H56D'],
      metric: 'negative_count',
      timeWindow: '3d',
      condition: 'increase',
      threshold: '20'
    }
  },
  {
    priority: '中',
    title: 'H77扶手屏异常监控',
    description: '用户反馈量稳定上升，建议关注',
    volume: '89次',
    trend: '+10%',
    risk: '中',
    config: {
      target: ['秦山', 'H77A'],
      metric: 'negative_count',
      timeWindow: '7d',
      condition: 'increase',
      threshold: '15'
    }
  },
  {
    priority: '中',
    title: 'H37尾灯水雾问题',
    description: '季节性问题，建议持续监控',
    volume: '67次',
    trend: '+5%',
    risk: '中',
    config: {
      target: ['知音', 'H37A'],
      metric: 'negative_count',
      timeWindow: '7d',
      condition: 'increase',
      threshold: '10'
    }
  }
]);

// 车型级联选择器配置
const carModelOptions = ref([
  {
    value: "FREE",
    label: "①FREE",
    children: [
      { value: "H97E", label: "H97E" },
      { value: "H97D", label: "H97D" },
      { value: "H97C", label: "H97C" },
      { value: "H97B", label: "H97B" },
      { value: "H97A", label: "H97A" },
    ],
  },
  {
    value: "追光",
    label: "②追光",
    children: [
      { value: "H53A", label: "H53A" },
      { value: "H53B", label: "H53B" },
    ],
  },
  {
    value: "知音",
    label: "③知音",
    children: [
      { value: "H37A", label: "H37A" },
      { value: "H37B", label: "H37B" },
    ],
  },
  {
    value: "梦想家",
    label: "④梦想家",
    children: [
      { value: "H56D", label: "H56D" },
      { value: "H56C", label: "H56C" },
      { value: "H56B", label: "H56B" },
      { value: "H56A", label: "H56A" },
    ],
  },
  {
    value: "秦山",
    label: "⑤秦山",
    children: [{ value: "H77A", label: "H77A" }],
  },
]);

const ruleForm = ref<{
  target: string[];
  metric: string;
  timeWindow: string;
  condition: string;
  threshold: string;
}>({
  target: [],
  metric: "",
  timeWindow: "3d",
  condition: "increase",
  threshold: "20",
});

const parseNaturalLanguage = () => {
  if (!naturalLanguage.value.trim()) {
    ElMessage.warning("请输入监控规则描述");
    return;
  }

  ElMessage.success("AI正在解析您的规则...");
  setTimeout(() => {
    createMethod.value = "form";
    ruleForm.value = {
      target: ["梦想家", "H56D"],
      metric: "negative_count",
      timeWindow: "3d",
      condition: "increase",
      threshold: "20",
    };
    ElMessage.success("解析完成！请确认规则配置");
  }, 1000);
};

const createRule = () => {
  // 表单验证
  if (createMethod.value === 'form') {
    // 验证监控对象
    if (!ruleForm.value.target || ruleForm.value.target.length === 0) {
      ElMessage.warning('请选择监控对象');
      return;
    }
    
    // 验证监控指标
    if (!ruleForm.value.metric) {
      ElMessage.warning('请选择监控指标');
      return;
    }
    
    // 验证阈值
    if (!ruleForm.value.threshold || ruleForm.value.threshold === '') {
      ElMessage.warning('请输入阈值');
      return;
    }
    
    // 验证阈值是否为有效数字
    const thresholdNum = parseFloat(ruleForm.value.threshold);
    if (isNaN(thresholdNum) || thresholdNum <= 0 || thresholdNum > 100) {
      ElMessage.warning('请输入1-100之间的有效阈值');
      return;
    }
  } else if (createMethod.value === 'natural') {
    // 自然语言模式：检查是否已解析
    if (!naturalLanguage.value.trim()) {
      ElMessage.warning('请输入监控规则描述');
      return;
    }
    // 检查是否点击了AI解析
    if (!ruleForm.value.target || ruleForm.value.target.length === 0) {
      ElMessage.warning('请先点击"AI解析"按钮');
      return;
    }
  }
  
  // 构建规则描述（用于显示）
  const targetDesc = ruleForm.value.target.join(' / ');
  const metricDesc = ruleForm.value.metric === 'negative_count' ? '负面反馈' : '正面反馈';
  const conditionDesc = ruleForm.value.condition === 'increase' ? '增长' : '减少';
  const timeWindowDesc = ruleForm.value.timeWindow === '1d' ? '1天' : 
                         ruleForm.value.timeWindow === '3d' ? '3天' : 
                         ruleForm.value.timeWindow === '7d' ? '7天' : '30天';
  
  // 生成任务名称
  const taskName = `${targetDesc} ${metricDesc}监控`;
  
  // 添加到任务列表（添加到数组开头）
  monitoringTasks.value.unshift({
    name: taskName,
    status: 'running',
    frequency: '每小时',
    lastCheck: '刚刚',
    triggerCount: 0
  });
  
  // 显示成功消息
  ElMessage({
    type: 'success',
    message: `监控规则已创建！\n监控对象: ${targetDesc}\n触发条件: ${metricDesc}${timeWindowDesc}内${conditionDesc}超过${ruleForm.value.threshold}%`,
    duration: 3000,
    showClose: true
  });
  
  // 重置表单
  ruleForm.value = {
    target: [],
    metric: "",
    timeWindow: "3d",
    condition: "increase",
    threshold: "20",
  };
  naturalLanguage.value = "";
  createMethod.value = "natural";
  
  // 关闭对话框
  showCreateDialog.value = false;
};

// 编辑任务
const editTask = (taskName: string) => {
  ElMessage.info(`正在编辑任务：${taskName}`);
  showCreateDialog.value = true;
};

// 暂停/启用任务
const pauseTask = (taskName: string) => {
  ElMessage({
    type: "success",
    message: `任务"${taskName}"已暂停`,
  });
};

// 删除任务
const deleteTask = (taskName: string) => {
  ElMessageBox.confirm(
    `确定要删除监控任务"${taskName}"吗？`,
    "删除确认",
    {
      type: "warning",
      confirmButtonText: "确定",
      cancelButtonText: "取消",
    }
  )
    .then(() => {
      // 从任务列表中删除
      const index = monitoringTasks.value.findIndex(task => task.name === taskName);
      if (index !== -1) {
        monitoringTasks.value.splice(index, 1);
      }
      ElMessage.success(`任务"${taskName}"已删除`);
    })
    .catch(() => {
      ElMessage.info("已取消删除");
    });
};

// 选择推荐的监控规则
const selectRecommendation = (rec: any) => {
  selectedRecommendation.value = rec;
  // 可以添加视觉反馈
};

// 从推荐创建监控规则
const createFromRecommendation = () => {
  if (!selectedRecommendation.value) {
    ElMessage.warning('请先选择一个推荐规则');
    return;
  }
  
  // 填充表单数据
  ruleForm.value = selectedRecommendation.value.config;
  
  // 关闭推荐对话框，打开创建对话框
  showRecommendDialog.value = false;
  showCreateDialog.value = true;
  createMethod.value = 'form';
  
  ElMessage.success('已加载推荐规则，请确认后保存');
};
</script>

<style scoped>
.monitoring-page {
  min-height: 100%;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
}

.stat-card {
  padding: 30px;
  border-radius: 8px;
  text-align: center;
  color: #fff;
  transition: transform 0.3s;
}

.stat-card:hover {
  transform: translateY(-5px);
}

.stat-card.running {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.stat-card.triggered {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.stat-card.disabled {
  background: linear-gradient(135deg, #a8a8a8 0%, #636363 100%);
}

.stat-number {
  font-size: 48px;
  font-weight: bold;
  margin-bottom: 10px;
}

.stat-label {
  font-size: 16px;
  opacity: 0.9;
}

.alert-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

.alert-content {
  flex: 1;
}

.alert-content h4 {
  margin: 0 0 8px 0;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
}

.alert-actions {
  display: flex;
  gap: 10px;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-weight: 500;
}

.card-header > div {
  display: flex;
  align-items: center;
  gap: 8px;
}

.task-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.task-item {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 16px;
  background: #f5f7fa;
  border-radius: 8px;
  transition: all 0.3s;
}

.task-item:hover {
  background: #e8eaf0;
  transform: translateX(4px);
}

.task-status {
  flex-shrink: 0;
}

.task-info {
  flex: 1;
}

.task-info h4 {
  margin: 0 0 8px 0;
  font-size: 15px;
  color: #303133;
}

.task-meta {
  display: flex;
  gap: 20px;
  font-size: 13px;
  color: #909399;
}

.task-actions {
  flex-shrink: 0;
}

.natural-input {
  padding: 10px 0;
}

/* 智能推荐样式 */
.recommend-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-height: 400px;
  overflow-y: auto;
}

.recommend-item {
  position: relative;
  padding: 16px;
  border: 2px solid #E4E7ED;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.recommend-item:hover {
  border-color: #409EFF;
  background: #f0f9ff;
  transform: translateX(4px);
}

.recommend-item.selected {
  border-color: #409EFF;
  background: #ecf5ff;
  box-shadow: 0 2px 12px rgba(64, 158, 255, 0.3);
}

.recommend-item.selected::before {
  content: '✓';
  position: absolute;
  right: 12px;
  top: 12px;
  width: 24px;
  height: 24px;
  background: #409EFF;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}



.recommend-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 10px;
}

.recommend-title {
  font-size: 16px;
  font-weight: 600;
  color: #303133;
}

.recommend-desc {
  font-size: 14px;
  color: #606266;
  margin-bottom: 12px;
  line-height: 1.6;
}

.recommend-stats {
  display: flex;
  gap: 20px;
  font-size: 13px;
  color: #909399;
}

.recommend-stats span {
  display: flex;
  align-items: center;
  gap: 4px;
}

.recommend-stats strong {
  color: #303133;
}
</style>
