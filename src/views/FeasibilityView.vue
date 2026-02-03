<template>
  <div class="feasibility-page">
    <!-- 输入区域 -->
    <el-card class="input-card" shadow="never">
      <div class="input-wrapper">
        <el-icon class="input-icon"><Magic /></el-icon>
        <input 
          v-model="userInput"
          class="custom-input" 
          placeholder="输入需求关键词进行可行性分析，如'智能座舱语音交互升级'"
          @keyup.enter="generateReport"
        />
        <el-button type="primary" class="generate-btn" @click="generateReport" :loading="isGenerating">
          <el-icon v-if="!isGenerating"><DocumentAdd /></el-icon>
          {{ isGenerating ? 'AI生成中...' : '生成报告' }}
        </el-button>
      </div>
    </el-card>

    <!-- 生成中的加载动画 -->
    <div v-if="isGenerating" class="generating-animation">
      <el-icon class="is-loading" :size="50"><Loading /></el-icon>
      <p class="generating-text">AI正在基于VOC数据生成可行性报告...</p>
      <el-progress :percentage="generatingProgress" :stroke-width="8" />
    </div>

    <!-- 报告内容 -->
    <div v-if="reportGenerated" class="report-container">
      <!-- 报告头部 -->
      <el-card class="report-header" shadow="hover">
        <div class="report-title-section">
          <h1 class="report-title">{{ reportData.title }}</h1>
          <div class="report-meta">
            <el-tag type="info" size="large">自动生成报告</el-tag>
            <span class="report-time">生成时间：{{ currentTime }}</span>
          </div>
        </div>
      </el-card>

      <!-- 评分卡片 -->
      <el-card class="score-card" shadow="hover">
        <div class="score-section">
          <div class="total-score">
            <div class="score-label">综合评分</div>
            <div class="score-value" :class="'level-' + reportData.score.level">
              {{ animatedScore }}
            </div>
            <div class="score-level">
              <el-tag :type="getLevelType(reportData.score.level)" size="large">
                {{ reportData.score.levelText }}
              </el-tag>
            </div>
          </div>
          <div class="score-breakdown">
            <div class="breakdown-item">
              <span class="breakdown-label">用户价值</span>
              <el-progress :percentage="(reportData.score.userValue / 40) * 100" :format="() => reportData.score.userValue" />
            </div>
            <div class="breakdown-item">
              <span class="breakdown-label">市场机会</span>
              <el-progress :percentage="(reportData.score.marketOpportunity / 30) * 100" :format="() => reportData.score.marketOpportunity" color="#E6A23C" />
            </div>
            <div class="breakdown-item">
              <span class="breakdown-label">技术可行性</span>
              <el-progress :percentage="(reportData.score.technicalFeasibility / 20) * 100" :format="() => reportData.score.technicalFeasibility" color="#67C23A" />
            </div>
            <div class="breakdown-item">
              <span class="breakdown-label">战略匹配</span>
              <el-progress :percentage="(reportData.score.strategicAlignment / 10) * 100" :format="() => reportData.score.strategicAlignment" color="#909399" />
            </div>
          </div>
        </div>
      </el-card>

      <!-- Tab切换 -->
      <el-tabs v-model="activeTab" type="card" class="output-tabs">
        <!-- Tab1: 完整报告 -->
        <el-tab-pane label="完整报告（PDF）" name="report">
          <el-card class="report-content" shadow="hover">
        <div class="report-body">
          <!-- 第一章 -->
          <div class="chapter">
            <h2 class="chapter-title">
              <el-icon><Document /></el-icon>
              第一章：需求概况
            </h2>
            <div v-for="section in reportData.chapters[0].sections" :key="section.subtitle" class="section">
              <h3 class="section-subtitle">{{ section.subtitle }}</h3>
              <div class="section-content" v-html="formatContent(section.content)"></div>
            </div>
          </div>

          <!-- 第二章 -->
          <div class="chapter">
            <h2 class="chapter-title">
              <el-icon><Document /></el-icon>
              {{ reportData.chapters[1].title }}
            </h2>
            
            <!-- 2.1 需求声量与趋势 -->
            <div class="section">
              <h3 class="section-subtitle">{{ reportData.chapters[1].sections[0].subtitle }}</h3>
              <div class="section-content" v-html="formatContent(reportData.chapters[1].sections[0].content)"></div>
              <div class="chart-container">
                <el-row :gutter="16">
                   <el-col :span="12">
                      <div ref="volumeTrendChart" style="height: 350px;"></div>
                   </el-col>
                   <el-col :span="12">
                      <div ref="sourceChart" style="height: 350px;"></div>
                   </el-col>
                </el-row>
              </div>
            </div>

            <!-- 2.2 用户画像与痛点 -->
            <div class="section">
              <h3 class="section-subtitle">{{ reportData.chapters[1].sections[1].subtitle }}</h3>
              <div class="section-content" v-html="formatContent(reportData.chapters[1].sections[1].content)"></div>
              <div class="chart-container">
                <el-row :gutter="16">
                  <el-col :span="12">
                    <div ref="sentimentChart" style="height: 300px;"></div>
                  </el-col>
                  <el-col :span="12">
                    <div ref="ageChart" style="height: 300px;"></div>
                  </el-col>
                </el-row>
              </div>
            </div>
          </div>

          <!-- 第三章 -->
          <div class="chapter">
            <h2 class="chapter-title">
              <el-icon><Document /></el-icon>
              {{ reportData.chapters[2].title }}
            </h2>
            
            <!-- 3.1 -->
            <div class="section">
              <h3 class="section-subtitle">{{ reportData.chapters[2].sections[0].subtitle }}</h3>
              <div class="section-content" v-html="formatContent(reportData.chapters[2].sections[0].content)"></div>
            </div>

            <!-- 3.2 -->
            <div class="section">
              <h3 class="section-subtitle">{{ reportData.chapters[2].sections[1].subtitle }}</h3>
              <div class="section-content" v-html="formatContent(reportData.chapters[2].sections[1].content)"></div>
              <div class="chart-container">
                <div ref="competitorRadarChart" style="height: 400px;"></div>
              </div>
            </div>

            <!-- 3.3 (Optional) -->
            <div v-if="reportData.chapters[2].sections[2]" class="section">
              <h3 class="section-subtitle">{{ reportData.chapters[2].sections[2].subtitle }}</h3>
              <div class="section-content" v-html="formatContent(reportData.chapters[2].sections[2].content)"></div>
            </div>
          </div>

          <!-- 第四章及其他 -->
          <div v-for="chapter in reportData.chapters.slice(3)" :key="chapter.id" class="chapter">
            <h2 class="chapter-title">
              <el-icon><Document /></el-icon>
              {{ chapter.title }}
            </h2>
            <div v-for="section in chapter.sections" :key="section.subtitle" class="section">
              <h3 class="section-subtitle">{{ section.subtitle }}</h3>
              <div class="section-content" v-html="formatContent(section.content)"></div>
            </div>
          </div>
        </div>

        <!-- 操作按钮 -->
        <div class="report-actions">
          <el-button type="primary" @click="exportPDF">
            <el-icon><Download /></el-icon>
            导出完整报告（PDF）
          </el-button>
        </div>
          </el-card>
        </el-tab-pane>

        <!-- Tab2: 执行摘要 -->
        <el-tab-pane label="执行摘要（一页纸）" name="summary">
          <el-card class="summary-card" shadow="hover">
        <template #header>
          <div class="card-header">
            <el-icon><Document /></el-icon>
            <span>执行摘要（一页纸决策参考）</span>
          </div>
        </template>
        <div class="executive-summary">
          <div class="summary-header">
            <h3>{{ reportData.title }}</h3>
            <el-tag :type="getLevelType(reportData.score.level)" size="large">
              {{ reportData.score.level }} - {{ reportData.score.levelText }}
            </el-tag>
          </div>
          
          <el-row :gutter="20" class="summary-scores">
            <el-col :span="6">
              <div class="score-item">
                <div class="score-num">{{ reportData.score.userValue }}</div>
                <div class="score-label">用户价值分</div>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="score-item">
                <div class="score-num">{{ reportData.score.marketOpportunity }}</div>
                <div class="score-label">市场机会分</div>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="score-item">
                <div class="score-num">{{ reportData.score.technicalFeasibility }}</div>
                <div class="score-label">技术可行性分</div>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="score-item">
                <div class="score-num">{{ reportData.score.strategicAlignment }}</div>
                <div class="score-label">战略匹配分</div>
              </div>
            </el-col>
          </el-row>
          
          <div class="summary-conclusion">
            <h4>核心结论</h4>
            <ul>
              <li>近90天声量环比增长<strong>42%</strong>，正面期待率达<strong>58%</strong></li>
              <li>竞品搭载率<strong>45%</strong>但满意度仅<strong>3.2分</strong>，存在显著改进机会</li>
              <li>技术复杂度<strong>中等</strong>，需整合3个现有系统</li>
              <li>与公司战略匹配度<strong>89%</strong>，符合智能化升级路线</li>
            </ul>
          </div>
          
          <div class="summary-action">
            <h4>建议行动</h4>
            <p>{{ reportData.score.recommendation }}</p>
          </div>
        </div>
          </el-card>
        </el-tab-pane>

        <!-- Tab3: 优先级建议与风险提示 -->
        <el-tab-pane label="优先级与风险" name="risk">
          <el-card class="risk-card" shadow="hover">
        <template #header>
          <div class="card-header">
            <el-icon><Warning /></el-icon>
            <span>优先级建议与风险提示</span>
          </div>
        </template>
        
        <div class="priority-section">
          <h4>优先级评定</h4>
          <div class="priority-result">
            <div class="priority-badge" :class="'priority-' + reportData.score.level.toLowerCase()">
              {{ reportData.score.level }}
            </div>
            <div class="priority-info">
              <div class="priority-text">{{ reportData.score.levelText }}</div>
              <div class="priority-score">综合评分：{{ reportData.score.total }}分</div>
            </div>
          </div>
          <div class="priority-formula">
            评分公式：0.4×{{ reportData.score.userValue }} + 0.3×{{ reportData.score.marketOpportunity }} + 0.2×{{ reportData.score.technicalFeasibility }} + 0.1×{{ reportData.score.strategicAlignment }} = {{ reportData.score.total }}分
          </div>
        </div>

        <el-divider />

        <div class="risk-section">
          <h4>风险提示</h4>
          <el-alert
            title="中等风险：第三方供应商依赖"
            type="warning"
            :closable="false"
            show-icon
            style="margin-bottom: 12px;"
          >
            语音识别服务依赖外部供应商（如科大讯飞），存在数据安全和供应稳定性风险。建议多家供应商备选。
          </el-alert>
          <el-alert
            title="中等风险：低配车型性能影响"
            type="warning"
            :closable="false"
            show-icon
            style="margin-bottom: 12px;"
          >
            多轮对话功能需占用更多系统内存，可能对低配车型体验产生负面影响。建议差异化配置方案。
          </el-alert>
          <el-alert
            title="低风险：用户接受度"
            type="success"
            :closable="false"
            show-icon
          >
            基于声量分析，用户期待值高（58%正面），接受度风险低。
          </el-alert>
        </div>

        <el-divider />

        <div class="action-section">
          <h4>下一步行动建议</h4>
          <el-timeline>
            <el-timeline-item timestamp="Q2 2026" placement="top" color="#409EFF">
              完成技术预研和供应商评估
            </el-timeline-item>
            <el-timeline-item timestamp="Q3 2026" placement="top" color="#67C23A">
              完成详细设计和开发排期
            </el-timeline-item>
            <el-timeline-item timestamp="Q4 2026" placement="top" color="#E6A23C">
              启动开发并同步进行用户测试
            </el-timeline-item>
            <el-timeline-item timestamp="Q1 2027" placement="top" color="#F56C6C">
              功能上线并持续优化
            </el-timeline-item>
          </el-timeline>
        </div>
          </el-card>
        </el-tab-pane>

        <!-- Tab3: 成本分析 -->
        <el-tab-pane label="成本分析" name="cost">
          <el-card class="cost-analysis-card" shadow="hover">
            <!-- 成本分析标题 -->
            <div class="cost-header">
              <h2>🚀 智能成本快算结果 - 音响系统升级</h2>
              <el-tag type="success">计算耗时: 2.8秒</el-tag>
            </div>

            <!-- 需求基本信息 -->
            <el-card class="info-section" shadow="never">
              <div class="info-grid">
                <div class="info-item">
                  <span class="info-label">需求名称：</span>
                  <span class="info-value">岚图梦想家音响系统升级（10喇叭→20喇叭）</span>
                </div>
                <div class="info-item">
                  <span class="info-label">升级内容：</span>
                  <span class="info-value">增加喇叭数量，升级品牌至哈曼卡顿，优化声场</span>
                </div>
                <div class="info-item">
                  <span class="info-label">车型适配：</span>
                  <span class="info-value">岚图梦想家 全系（标配/选配）</span>
                </div>
                <div class="info-item">
                  <span class="info-label">预估年销量：</span>
                  <span class="info-value">150,000台</span>
                </div>
                <div class="info-item">
                  <span class="info-label">影响用户：</span>
                  <span class="info-value">全系用户</span>
                </div>
              </div>
            </el-card>

            <!-- 成本总览 -->
            <div class="cost-overview-section">
              <h3 class="section-title">📊 成本总览（单台分摊）</h3>
              <div class="cost-cards">
                <el-card class="cost-item-card rd-cost">
                  <div class="cost-type">研发成本</div>
                  <div class="cost-amount">¥2.80</div>
                  <div class="cost-desc">(按5年摊销，¥150,000台/年)</div>
                </el-card>
                <el-card class="cost-item-card material-cost">
                  <div class="cost-type">物料成本</div>
                  <div class="cost-amount">¥1,680.00</div>
                  <div class="cost-desc">(喇叭+功放+线材+隔音)</div>
                </el-card>
                <el-card class="cost-item-card compute-cost">
                  <div class="cost-type">其他成本</div>
                  <div class="cost-amount">¥35.00</div>
                  <div class="cost-desc">(调音+认证+生产调整)</div>
                </el-card>
              </div>

              <!-- 成本汇总 -->
              <el-card class="total-cost-summary">
                <div class="summary-item total">
                  <span>🔴 单台总成本：</span>
                  <strong>¥1,717.80（不含税）</strong>
                </div>
                <div class="summary-item">
                  <span>🟢 建议售价：</span>
                  <span>标配+¥5,000，选配+¥8,000</span>
                </div>
                <div class="summary-item">
                  <span>📈 毛利率：</span>
                  <span>标配74.0%，选配78.5%</span>
                </div>
                <div class="summary-item">
                  <span>⚠️ 投资回收：</span>
                  <span>预计选装率提升20%，总销量提升5%</span>
                </div>
              </el-card>
            </div>

            <!-- 成本构成详情 -->
            <div class="cost-details-section">
              <h3 class="section-title">成本构成详情</h3>

              <!-- 一、研发成本明细 -->
              <el-card class="detail-card">
                <h4>一、研发成本明细 ¥2.80</h4>
                <el-table :data="rdCostDetails" border>
                  <el-table-column prop="item" label="项目" width="200" />
                  <el-table-column prop="calculation" label="计算" />
                  <el-table-column prop="amount" label="金额" width="150" align="right" />
                </el-table>
                <div class="detail-note">
                  <el-icon><InfoFilled /></el-icon>
                  [📈 详细依据：参考理想L9音响升级项目]
                </div>
              </el-card>

              <!-- 二、物料成本明细 -->
              <el-card class="detail-card">
                <h4>二、物料成本明细 ¥1,680.00</h4>
                <el-table :data="materialCostDetails" border>
                  <el-table-column prop="item" label="物料项" width="250" />
                  <el-table-column prop="spec" label="规格说明" />
                  <el-table-column prop="price" label="单价" width="120" align="right" />
                </el-table>
                <div class="detail-note">
                  <el-icon><InfoFilled /></el-icon>
                  批量折扣：采购150,000套，单价下降15%
                  <el-link type="primary" style="margin-left: 10px;">🔗 供应商报价单：哈曼卡顿-2024-0425</el-link>
                </div>
              </el-card>

              <!-- 三、其他成本明细 -->
              <el-card class="detail-card">
                <h4>三、其他成本明细 ¥35.00</h4>
                <el-table :data="otherCostDetails" border>
                  <el-table-column prop="item" label="费用项" width="200" />
                  <el-table-column prop="description" label="说明" />
                  <el-table-column prop="price" label="单价" width="120" align="right" />
                </el-table>
                <div class="total-yearly">
                  小计：¥35.00/台
                </div>
                <div class="detail-note optimization">
                  <el-icon><Warning /></el-icon>
                  [⚡ 批量效应：单台成本降低16.7%]
                </div>
              </el-card>
            </div>

            <!-- 竞品对标与市场压力 -->
            <div class="competitor-section">
              <h3 class="section-title">竞品对标与市场压力</h3>
              <el-card class="detail-card">
                <el-table :data="competitorData" border>
                  <el-table-column prop="brand" label="品牌车型" width="150" />
                  <el-table-column prop="speakers" label="喇叭配置" />
                  <el-table-column prop="status" label="标配/选配" width="120" />
                  <el-table-column prop="price" label="价格" width="120" />
                </el-table>
                <div class="detail-note">
                  <el-icon><InfoFilled /></el-icon>
                  市场压力：作为MPV，音响是核心体验差距点。机会窗口：可与影院模式、K歌功能形成生态
                </div>
              </el-card>
            </div>

            <!-- 投资回报分析 -->
            <div class="roi-section">
              <h3 class="section-title">投资回报分析</h3>
              
              <el-card class="roi-detail-card">
                <h4>� 直接收益</h4>
                <ul class="benefit-list">
                  <li>预计60%用户选择标配（+¥5,000）：¥4.5亿/年</li>
                  <li>预计25%用户选择选配（+¥8,000）：¥3.0亿/年</li>
                  <li>不选用户：15%</li>
                  <li><strong>年总增收：¥7.5亿</strong></li>
                </ul>
              </el-card>

              <el-card class="roi-detail-card" style="margin-top: 16px;">
                <h4>💰 成本分析</h4>
                <ul class="benefit-list">
                  <li>研发一次性投入：¥43.56万</li>
                  <li>年总成本：¥2.5767亿（150,000台×¥1,717.80）</li>
                  <li>年毛利润：¥4.9233亿</li>
                  <li>毛利率：65.6%</li>
                  <li>净利润率（估算）：约45-50%</li>
                </ul>
              </el-card>

              <el-row :gutter="20" style="margin-top: 20px;">
                <el-col :span="12">
                  <el-card class="roi-card">
                    <h4>⏱️ 投资回收期</h4>
                    <div class="payback-info">
                      <div>• 研发投入回收期：<strong>小于1个月</strong></div>
                      <div>• 项目整体投资回报期：<strong>6.3个月</strong></div>
                      <div>• 5年总利润：<strong>约24.6亿</strong></div>
                    </div>
                  </el-card>
                </el-col>
                <el-col :span="12">
                  <el-card class="roi-card strategic">
                    <h4>🎯 战略价值</h4>
                    <div class="star-rating">★★★★★（5星）</div>
                    <ul class="value-list">
                      <li>✅ 补齐MPV核心体验短板</li>
                      <li>✅ 提升品牌高端形象</li>
                      <li>✅ 形成座舱娱乐生态闭环</li>
                    </ul>
                  </el-card>
                </el-col>
              </el-row>
            </div>

            <!-- 快速决策建议 -->
            <div class="decision-section">
              <h3 class="section-title">快速决策建议</h3>
              
              <el-card class="decision-card">
                <div class="feasibility-checks">
                  <el-tag type="success" size="large"><el-icon><Check /></el-icon> 成本可行性：优秀（单车成本<¥1,800，毛利率>65%）</el-tag>
                  <el-tag type="success" size="large"><el-icon><Check /></el-icon> 技术可行性：成熟（哈曼卡顿方案已验证）</el-tag>
                  <el-tag type="warning" size="large"><el-icon><Warning /></el-icon> 市场紧迫性：高（现款音响是主要吐槽点）</el-tag>
                </div>

                <el-divider />

                <h4>🎯 建议决策</h4>
                
                <div class="solution-options">
                  <!-- 方案A -->
                  <el-card class="solution-card recommended">
                    <div class="solution-header">
                      <span class="solution-badge gold">🥇</span>
                      <h5>推荐方案A：哈曼卡顿20喇叭全系升级</h5>
                    </div>
                    <ul class="solution-details">
                      <li>• 低配版标配，高配版可选配增强版</li>
                      <li>• 立即启动，目标2024年Q4量产</li>
                      <li>• 总投入：首年约¥2.5767亿（150,000台）</li>
                    </ul>
                  </el-card>

                  <!-- 方案B -->
                  <el-card class="solution-card">
                    <div class="solution-header">
                      <span class="solution-badge silver">🥈</span>
                      <h5>替代方案B：分品牌策略</h5>
                    </div>
                    <ul class="solution-details">
                      <li>• 梦想家高配：哈曼卡顿20喇叭</li>
                      <li>• 梦想家低配：普通品牌16喇叭</li>
                      <li>• 成本差异：低配成本下降¥650/台</li>
                    </ul>
                  </el-card>
                </div>

                <!-- 操作按钮 -->
                <div class="action-buttons">
                  <el-button type="default" @click="recalculate">🔄 重新计算</el-button>
                  <el-button type="primary" @click="generateDetailedReport">📊 生成详细报告</el-button>
                  <el-button type="success" @click="confirmPlan">✅ 确认方案A并立项</el-button>
                </div>
              </el-card>
            </div>
          </el-card>
        </el-tab-pane>
      </el-tabs>
    </div>

    <!-- 初始空状态 -->
    <el-empty v-if="!reportGenerated && !isGenerating" description="输入需求关键词，AI将为您生成详细的可行性评估报告" :image-size="200" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick } from 'vue'
import * as echarts from 'echarts'
import feasibilityData from '../mock/feasibility.json'
import { ElMessage } from 'element-plus'

const userInput = ref('')
const isGenerating = ref(false)
const reportGenerated = ref(false)
const generatingProgress = ref(0)
const animatedScore = ref(0)
const activeTab = ref('report')

// 图表refs
const volumeTrendChart = ref<HTMLElement>()
const sentimentChart = ref<HTMLElement>()
const sourceChart = ref<HTMLElement>()
const ageChart = ref<HTMLElement>()
const incomeChart = ref<HTMLElement>()
const regionChart = ref<HTMLElement>()
const competitorRadarChart = ref<HTMLElement>()

const reportData = ref(feasibilityData.report)

// 成本分析数据
const rdCostDetails = ref([
  { item: '声学设计', calculation: '60人天 × ¥3,200', amount: '¥192,000' },
  { item: '调音匹配', calculation: '40人天 × ¥3,000', amount: '¥120,000' },
  { item: '集成测试', calculation: '30人天 × ¥2,800', amount: '¥84,000' },
  { item: '项目管理', calculation: '人力×10%', amount: '¥39,600' },
  { item: '小计', calculation: '¥435,600 ÷ 150,000台 ÷ 5年', amount: '¥2.80' }
])

const materialCostDetails = ref([
  { item: '哈曼卡顿喇叭单元（20个）', spec: '批量折扣15%', price: '¥1,050.00' },
  { item: '16通道独立功放', spec: '专业级功放', price: '¥440.00' },
  { item: '专用音频线束', spec: '屏蔽线材', price: '¥110.00' },
  { item: '隔音材料增强', spec: '高级隔音棉', price: '¥80.00' }
])

const otherCostDetails = ref([
  { item: '专业调音服务', description: '外包专业调音', price: '¥12.00' },
  { item: '哈曼卡顿授权费', description: '品牌授权', price: '¥15.00' },
  { item: '产线调整费', description: '摊销成本', price: '¥5.00' },
  { item: '质量检测', description: '音质检测', price: '¥3.00' }
])

const competitorData = ref([
  { brand: '理想L9', speakers: '21喇叭杜比全景声', status: '标配', price: '-' },
  { brand: '小鹏G9', speakers: '20喇叭丹拿', status: '选配', price: '+¥6,000' },
  { brand: '问界M7', speakers: '19喇叭华为SOUND', status: '标配', price: '-' },
  { brand: '腾势D9', speakers: '14喇叭丹拿', status: '标配', price: '-' },
  { brand: '岚图现款', speakers: '10喇叭普通品牌', status: '标配', price: '-' }
])

// 成本分析方法
const recalculate = () => {
  ElMessage.info('重新计算成本中...')
}

const generateDetailedReport = () => {
  ElMessage.success('正在生成详细PDF报告...')
}

const confirmPlan = () => {
  ElMessage.success('已确认方案A，正在创建项目立项申请...')
}

const currentTime = computed(() => {
  const now = new Date()
  return now.toLocaleString('zh-CN', { 
    year: 'numeric', 
    month: '2-digit', 
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
})

const generateReport = () => {
  if (!userInput.value.trim()) {
    ElMessage.warning('请输入需求关键词')
    return
  }

  isGenerating.value = true
  reportGenerated.value = false
  generatingProgress.value = 0
  animatedScore.value = 0
  
  // 动态更新标题，自动添加后缀
  reportData.value.title = userInput.value.endsWith('可行性评估报告') 
    ? userInput.value 
    : `${userInput.value}可行性评估报告`;

  // 模拟进度
  const progressInterval = setInterval(() => {
    generatingProgress.value += 10
    if (generatingProgress.value >= 100) {
      clearInterval(progressInterval)
    }
  }, 200)

  // 3秒后显示报告
  setTimeout(() => {
    isGenerating.value = false
    reportGenerated.value = true
    
    // 分数动画
    const scoreInterval = setInterval(() => {
      if (animatedScore.value < reportData.value.score.total) {
        animatedScore.value++
      } else {
        clearInterval(scoreInterval)
      }
    }, 20)
    
    // 初始化图表
    nextTick(() => {
      initCharts()
    })
  }, 3000)
}

const formatContent = (content: string) => {
  return content
    .replace(/\n/g, '<br />')
    .replace(/•/g, '<span style="color: #409EFF;">•</span>')
    .replace(/(\d+\.\d+)/g, '<strong>$1</strong>')
}

const getLevelType = (level: string) => {
  const types: Record<string, any> = {
    P0: 'success',
    P1: 'warning',
    P2: 'info',
    P3: 'danger',
  }
  return types[level] || 'info'
}

const exportPDF = () => {
  ElMessage.success('PDF导出功能开发中...')
}

const initCharts = () => {
  // 1. 声量趋势图
  if (volumeTrendChart.value) {
    const chart = echarts.init(volumeTrendChart.value)
    chart.setOption({
      title: { text: '近90天“音响系统”声量趋势', left: 'center' },
      tooltip: { trigger: 'axis' },
      xAxis: {
        type: 'category',
        data: ['第1周', '第2周', '第3周', '第4周', '第5周', '第6周', '第7周', '第8周', '第9周', '第10周', '第11周', '第12周']
      },
      yAxis: { type: 'value', name: '讨论量' },
      series: [{
        data: [150, 165, 180, 200, 240, 290, 350, 420, 500, 580, 650, 720],
        type: 'line',
        smooth: true,
        areaStyle: { color: 'rgba(102, 126, 234, 0.1)' },
        itemStyle: { color: '#667EEA' },
        name: '音响话题声量'
      }]
    })
  }

  // 2. 情感分布饼图
  if (sentimentChart.value) {
    const chart = echarts.init(sentimentChart.value)
    chart.setOption({
      title: { text: '用户评价情感分布', left: 'center' },
      tooltip: { trigger: 'item' },
      series: [{
        type: 'pie',
        radius: '60%',
        data: [
          { value: 45, name: '期待升级', itemStyle: { color: '#67C23A' } },
          { value: 35, name: '不满原车', itemStyle: { color: '#F56C6C' } },
          { value: 20, name: '观望/中性', itemStyle: { color: '#909399' } }
        ],
        label: { formatter: '{b}\n{d}%' }
      }]
    })
  }

  // 3. 声量来源分布
  if (sourceChart.value) {
    const chart = echarts.init(sourceChart.value)
    chart.setOption({
      title: { text: '声量来源', left: 'center' },
      tooltip: { trigger: 'item' },
      series: [{
        type: 'pie',
        radius: ['40%', '70%'],
        data: [
          { value: 55, name: '汽车垂直社区', itemStyle: { color: '#E6A23C' } },
          { value: 25, name: '社交媒体(抖音/红书)', itemStyle: { color: '#409EFF' } },
          { value: 15, name: '车主群/论坛', itemStyle: { color: '#67C23A' } },
          { value: 5, name: '客服/其他', itemStyle: { color: '#909399' } }
        ],
        label: { formatter: '{b}\n{d}%' }
      }]
    })
  }

  // 4. 年龄分布
  if (ageChart.value) {
    const chart = echarts.init(ageChart.value)
    chart.setOption({
      title: { text: '关注用户年龄', left: 'center' },
      tooltip: {},
      xAxis: { type: 'category', data: ['<25', '25-30', '30-40', '40+'] },
      yAxis: { type: 'value', name: '占比%' },
      series: [{
        data: [10, 20, 55, 15],
        type: 'bar',
        itemStyle: { color: '#5470C6' },
        name: '用户占比'
      }]
    })
  }

  // 5. 收入分布
  if (incomeChart.value) {
    const chart = echarts.init(incomeChart.value)
    chart.setOption({
      title: { text: '家庭年收入', left: 'center' },
      tooltip: {},
      xAxis: { type: 'category', data: ['20-30万', '30-50万', '50-80万', '80万+'] },
      yAxis: { type: 'value', name: '占比%' },
      series: [{
        data: [12, 45, 32, 11],
        type: 'bar',
        itemStyle: { color: '#91CC75' }
      }]
    })
  }

  // 6. 地区分布
  if (regionChart.value) {
    const chart = echarts.init(regionChart.value)
    chart.setOption({
      title: { text: '地区分布', left: 'center' },
      tooltip: { trigger: 'item' },
      series: [{
        type: 'pie',
        radius: '60%',
        data: [
          { value: 35, name: '华东', itemStyle: { color: '#5470C6' } },
          { value: 25, name: '华北', itemStyle: { color: '#91CC75' } },
          { value: 20, name: '华南', itemStyle: { color: '#FAC858' } },
          { value: 12, name: '西南', itemStyle: { color: '#EE6666' } },
          { value: 8, name: '其他', itemStyle: { color: '#73C0DE' } }
        ]
      }]
    })
  }

  // 7. 竞品听感对比雷达图
  if (competitorRadarChart.value) {
    const chart = echarts.init(competitorRadarChart.value)
    chart.setOption({
      title: { text: '竞品听感维度实测对比', left: 'center' },
      legend: { bottom: 0 },
      radar: {
        indicator: [
          { name: '高音解析', max: 5 },
          { name: '中频人声', max: 5 },
          { name: '低音下潜', max: 5 },
          { name: '声场广度', max: 5 },
          { name: '环绕沉浸', max: 5 }
        ]
      },
      series: [{
        name: '听感评分',
        type: 'radar',
        data: [
           {
            value: [4.2, 4.5, 4.0, 3.8, 4.0],
            name: '我方8单元(预期)',
            itemStyle: { color: '#409EFF' },
            areaStyle: { opacity: 0.3 }
          },
          {
            value: [3.8, 3.5, 4.2, 4.2, 4.2],
            name: '竞品14单元(均值)',
            itemStyle: { color: '#E6A23C' },
            areaStyle: { opacity: 0.3 }
          }
        ]
      }] 
    })
  }

}
</script>

<style scoped>
.feasibility-page {
  min-height: 100%;
}

.input-card {
  margin-bottom: 20px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
}

.input-icon {
  font-size: 24px;
  color: #fff;
}

.custom-input {
  flex: 1;
  border: none;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  font-size: 16px;
  padding: 12px 16px;
  border-radius: 6px;
  outline: none;
  color: #fff;
  transition: background 0.3s;
}

.custom-input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

.custom-input:focus {
  background: rgba(255, 255, 255, 0.3);
}

.generate-btn {
  padding: 12px 28px;
  background: #fff;
  color: #667eea;
  border: none;
  font-weight: bold;
}

.generate-btn:hover {
  background: #f0f0f0;
}

.generating-animation {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 20px;
  background: linear-gradient(135deg, #667eea22 0%, #764ba222 100%);
  border-radius: 12px;
  margin-bottom: 20px;
}

.generating-text {
  margin: 20px 0;
  font-size: 18px;
  color: #667eea;
  font-weight: 500;
}

.generating-animation :deep(.el-progress) {
  width: 60%;
  margin-top: 10px;
}

.report-header {
  margin-bottom: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.report-title-section {
  padding: 20px 0;
}

.report-title {
  font-size: 32px;
  margin: 0 0 15px 0;
  font-weight: bold;
}

.report-meta {
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 14px;
  opacity: 0.9;
}

.score-card {
  margin-bottom: 20px;
}

.score-section {
  display: flex;
  gap: 40px;
  align-items: center;
}

.total-score {
  flex-shrink: 0;
  text-align: center;
  padding: 20px;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  border-radius: 12px;
  color: #fff;
  min-width: 200px;
}

.score-label {
  font-size: 16px;
  margin-bottom: 10px;
  opacity: 0.9;
}

.score-value {
  font-size: 72px;
  font-weight: bold;
  line-height: 1;
  margin: 10px 0;
}

.score-level {
  margin-top: 15px;
}

.score-breakdown {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.breakdown-item {
  display: flex;
  align-items: center;
  gap: 15px;
}

.breakdown-label {
  min-width: 100px;
  font-size: 15px;
  color: #606266;
  font-weight: 500;
}

.breakdown-item :deep(.el-progress) {
  flex: 1;
}

.report-content {
  margin-bottom: 20px;
}

.report-body {
  max-width: 900px;
  margin: 0 auto;
}

.chapter {
  margin-bottom: 40px;
}

.chapter-title {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 24px;
  color: #303133;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #DCDFE6;
}

.section {
  margin-bottom: 25px;
}

.section-subtitle {
  font-size: 18px;
  color: #409EFF;
  margin-bottom: 12px;
  font-weight: 600;
}

.section-content {
  font-size: 15px;
  line-height: 1.8;
  color: #606266;
  padding-left: 10px;
}

.report-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 40px;
  padding-top: 30px;
  border-top: 1px solid #DCDFE6;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
}

.chart-container {
  margin-top: 20px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 8px;
}

/* Tab切换样式 */
.output-tabs {
  margin-top: 20px;
}

.output-tabs :deep(.el-tabs__header) {
  margin-bottom: 0;
}

.output-tabs :deep(.el-tabs__content) {
  padding-top: 0;
}

/* 执行摘要样式 */
.summary-card {
  margin-bottom: 20px;
}

.executive-summary .summary-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.executive-summary .summary-header h3 {
  margin: 0;
  font-size: 20px;
  color: #303133;
}

.summary-scores {
  margin-bottom: 24px;
}

.score-item {
  text-align: center;
  padding: 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  color: #fff;
}

.score-item .score-num {
  font-size: 32px;
  font-weight: bold;
}

.score-item .score-label {
  font-size: 12px;
  opacity: 0.9;
  margin-top: 4px;
}

.summary-conclusion,
.summary-action {
  margin-bottom: 20px;
}

.summary-conclusion h4,
.summary-action h4 {
  font-size: 16px;
  color: #303133;
  margin-bottom: 12px;
  border-left: 4px solid #667eea;
  padding-left: 12px;
}

.summary-conclusion ul {
  margin: 0;
  padding-left: 20px;
  color: #606266;
  line-height: 2;
}

.summary-conclusion strong {
  color: #667eea;
}

.summary-action p {
  color: #606266;
  line-height: 1.8;
  margin: 0;
}

/* 优先级与风险样式 */
.risk-card {
  margin-bottom: 20px;
}

.priority-section h4,
.risk-section h4,
.action-section h4 {
  font-size: 16px;
  color: #303133;
  margin-bottom: 16px;
  border-left: 4px solid #667eea;
  padding-left: 12px;
}

.priority-result {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 16px;
}

.priority-badge {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  font-weight: bold;
  color: #fff;
}

.priority-p0 {
  background: linear-gradient(135deg, #67C23A, #42b983);
}

.priority-p1 {
  background: linear-gradient(135deg, #E6A23C, #f39c12);
}

.priority-p2 {
  background: linear-gradient(135deg, #909399, #7f8c8d);
}

.priority-p3 {
  background: linear-gradient(135deg, #F56C6C, #e74c3c);
}

.priority-info .priority-text {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
}

.priority-info .priority-score {
  color: #909399;
  margin-top: 4px;
}

.priority-formula {
  background: #f5f7fa;
  padding: 12px 16px;
  border-radius: 4px;
  color: #606266;
  font-size: 14px;
}

.action-section :deep(.el-timeline) {
  padding-left: 10px;
}

/* 成本分析样式 */
.cost-analysis-card {
  margin-top: 16px;
}

.cost-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.cost-header h2 {
  margin: 0;
  font-size: 24px;
  color: #303133;
}

.info-section {
  background: #f5f7fa;
  margin-bottom: 24px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.info-item {
  font-size: 14px;
  line-height: 2;
}

.info-label {
  font-weight: bold;
  color: #606266;
}

.info-value {
  color: #303133;
}

.cost-overview-section {
  margin-bottom: 32px;
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 16px;
  padding-bottom: 8px;
  border-bottom: 2px solid #409EFF;
}

.cost-cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 16px;
}

.cost-item-card {
  text-align: center;
  padding: 24px !important;
}

.cost-item-card.rd-cost {
  border-left: 4px solid #409EFF;
}

.cost-item-card.material-cost {
  border-left: 4px solid #67C23A;
}

.cost-item-card.compute-cost {
  border-left: 4px solid #E6A23C;
}

.cost-type {
  font-size: 14px;
  color: #909399;
  margin-bottom: 12px;
}

.cost-amount {
  font-size: 28px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 8px;
}

.cost-desc {
  font-size: 12px;
  color: #909399;
}

.total-cost-summary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px !important;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
  font-size: 16px;
}

.summary-item.total {
  font-size: 20px;
  padding-bottom: 12px;
  border-bottom: 1px solid rgba(255,255,255,0.3);
  margin-bottom: 16px;
}

.summary-item strong {
  font-size: 24px;
}

.cost-details-section {
  margin-bottom: 32px;
}

.detail-card {
  margin-bottom: 20px;
}

.detail-card h4 {
  color: #303133;
  margin-bottom: 16px;
}

.detail-note {
  margin-top: 12px;
  padding: 8px 12px;
  background: #ecf5ff;
  border-left: 3px solid #409EFF;
  font-size: 13px;
  color: #606266;
  display: flex;
  align-items: center;
  gap: 8px;
}

.detail-note.optimization {
  background: #fef0f0;
  border-left-color: #F56C6C;
}

.total-yearly {
  text-align: right;
  font-weight: bold;
  margin-top: 12px;
  color: #303133;
}

.roi-section {
  margin-bottom: 32px;
}

.roi-card {
  height: 100%;
}

.roi-card h4 {
  margin-bottom: 16px;
  color: #303133;
}

.large-text {
  font-size: 20px;
  font-weight: bold;
  color: #909399;
  text-align: center;
  margin: 24px 0;
}

.benefit-list,
.value-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.benefit-list li,
.value-list li {
  padding: 8px 0;
  border-bottom: 1px dashed #E4E7ED;
}

.benefit-list li:last-child,
.value-list li:last-child {
  border-bottom: none;
}

.payback-info div {
  padding: 6px 0;
}

.roi-card.strategic {
  background: linear-gradient(135deg, #ffeaa7 0%, #fdcb6e 100%);
}

.star-rating {
  font-size: 24px;
  color: #F56C6C;
  text-align: center;
  margin: 16px 0;
}

.decision-section {
  margin-bottom: 24px;
}

.decision-card {
  padding: 24px !important;
}

.feasibility-checks {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

.feasibility-checks .el-tag {
  padding: 12px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.solution-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
  margin: 20px 0;
}

.solution-card {
  border: 2px solid #E4E7ED;
  transition: all 0.3s;
}

.solution-card:hover {
  border-color: #409EFF;
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.2);
}

.solution-card.recommended {
  border-color: #67C23A;
  background: linear-gradient(135deg, #f0fff4 0%, #c6f6d5 100%);
}

.solution-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.solution-badge {
  font-size: 24px;
}

.solution-badge.gold::after {
  content: '';
}

.solution-badge.silver::after {
  content: '';
}

.solution-header h5 {
  margin: 0;
  color: #303133;
}

.solution-details {
  list-style: none;
  padding: 0;
  margin: 0;
}

.solution-details li {
  padding: 4px 0;
  color: #606266;
}

.action-buttons {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 24px;
}

.action-buttons .el-button {
  min-width: 150px;
}
</style>
