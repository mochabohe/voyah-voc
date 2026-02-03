<template>
  <!-- 悬浮机器人图标 -->
  <div class="chatbot-wrapper">
    <el-badge
      :value="unreadCount"
      :hidden="unreadCount === 0"
      class="chatbot-badge"
    >
      <div class="chatbot-trigger" @click="toggleChat">
        <el-icon :size="32">
          <ChatDotRound />
        </el-icon>
      </div>
    </el-badge>

    <!-- 聊天弹窗 -->
    <transition name="slide-fade">
      <div v-show="isOpen" class="chatbot-dialog">
        <!-- 头部 -->
        <div class="chat-header">
          <div class="header-info">
            <el-avatar :size="36" style="background-color: #667eea">
              <el-icon><ChatDotRound /></el-icon>
            </el-avatar>
            <div class="header-text">
              <div class="header-title">VOC智能助手</div>
              <div class="header-status">
                <span class="status-dot"></span>
                在线
              </div>
            </div>
          </div>
          <div style="display: flex; gap: 12px; align-items: center;">
            <el-icon 
              class="new-chat-btn" 
              @click="startNewChat"
              title="新建会话"
            >
              <Plus />
            </el-icon>
            <el-icon class="close-btn" @click="toggleChat"><Close /></el-icon>
          </div>
        </div>

        <!-- 消息区域 -->
        <div class="chat-messages" ref="messagesContainer">
          <div
            v-for="(msg, index) in messages"
            :key="index"
            :class="['message', msg.role]"
          >
            <el-avatar
              v-if="msg.role === 'assistant'"
              :size="32"
              style="background-color: #667eea"
            >
              <el-icon><ChatDotRound /></el-icon>
            </el-avatar>
            <div class="message-content">
              <div class="message-bubble" v-html="msg.content" @click="handleMessageClick"></div>
              <div class="message-time">{{ msg.time }}</div>
            </div>
            <el-avatar
              v-if="msg.role === 'user'"
              :size="32"
              style="background-color: #409eff"
            >
              <el-icon><User /></el-icon>
            </el-avatar>
          </div>

          <!-- 打字中指示器 -->
          <div v-if="isTyping" class="message assistant">
            <el-avatar :size="32" style="background-color: #667eea">
              <el-icon><ChatDotRound /></el-icon>
            </el-avatar>
            <div class="typing-indicator">
              <span></span>
              <span></span>
              <span></span>
            </div>
          </div>
        </div>

        <!-- 快捷提问 -->
        <div v-if="messages.length === 1" class="quick-questions">
          <div
            v-for="(q, idx) in quickQuestions"
            :key="idx"
            class="quick-question"
            @click="sendMessage(q)"
          >
            {{ q }}
          </div>
        </div>

        <!-- 输入区域 -->
        <div class="chat-input">
          <el-input
            v-model="inputText"
            placeholder="输入您的问题..."
            @keyup.enter="handleSend"
            :disabled="isTyping"
          >
            <template #suffix>
              <el-button
                type="primary"
                :icon="isTyping ? 'Loading' : 'Promotion'"
                circle
                size="small"
                @click="handleSend"
                :disabled="!inputText.trim() || isTyping"
              />
            </template>
          </el-input>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, inject } from "vue";

// 使用 inject 获取导航方法
const navigate = inject('navigate') as (view: string, params?: any) => void;
const isOpen = ref(false);
const unreadCount = ref(1); // 有一条新推送
const inputText = ref("");
const isTyping = ref(false);
const messagesContainer = ref<HTMLElement>();

const quickQuestions = [
  "H77公域最关注问题top5",
  "H77核心竞品新闻",
  "H77近半年销量走势",
];

interface Message {
  role: "user" | "assistant";
  content: string;
  time: string;
}

// 构造竞品推送表格 HTML
const competitorTableHtml = `
<div style="font-weight:bold; margin-bottom:8px;">📢 竞品热点推送</div>
<table style="width:100%; border-collapse:collapse; font-size:12px; text-align:left; white-space:nowrap;">
  <tr style="background:#f0f2f5; color:#909399;">
    <th style="padding:4px;">热度</th>
    <th style="padding:4px;">话题名称</th>
    <th style="padding:4px; white-space:nowrap;">竞品</th>
    <th style="padding:4px;">声量</th>
    <th style="padding:4px;">情感</th>
  </tr>
  <tr 
    style="cursor:pointer; border-bottom:1px solid #ebeef5; transition:background 0.2s;" 
    data-action="navigate-competitor"
    data-topic="问界M9 OTA升级"
    onmouseover="this.style.background='#f0f9eb'" 
    onmouseout="this.style.background='white'"
  >
    <td style="padding:6px; color:#F56C6C;">🔥92</td>
    <td style="padding:6px; font-weight:bold; color:#409EFF;">问界M9 OTA升级...</td>
    <td style="padding:6px; white-space:nowrap;">问界</td>
    <td style="padding:6px;">5.2k</td>
    <td style="padding:6px; color:#67C23A;">😃85%</td>
  </tr>
  <tr style="border-bottom:1px solid #ebeef5;">
    <td style="padding:6px; color:#F56C6C;">🔥85</td>
    <td style="padding:6px;">小鹏G9城市NGP...</td>
    <td style="padding:6px; white-space:nowrap;">小鹏</td>
    <td style="padding:6px;">3.9k</td>
    <td style="padding:6px; color:#F56C6C;">😡78%</td>
  </tr>
  <tr style="border-bottom:1px solid #ebeef5;">
    <td style="padding:6px; color:#E6A23C;">🔥78</td>
    <td style="padding:6px;">问界M7冬季空...</td>
    <td style="padding:6px; white-space:nowrap;">问界</td>
    <td style="padding:6px;">2.8k</td>
    <td style="padding:6px; color:#F56C6C;">😡82%</td>
  </tr>
  <tr style="border-bottom:1px solid #ebeef5;">
    <td style="padding:6px; color:#E6A23C;">🔥72</td>
    <td style="padding:6px;">腾势D9第三排...</td>
    <td style="padding:6px; white-space:nowrap;">腾势</td>
    <td style="padding:6px;">2.3k</td>
    <td style="padding:6px; color:#F56C6C;">😡65%</td>
  </tr>
  <tr>
    <td style="padding:6px; color:#909399;">🔥68</td>
    <td style="padding:6px;">特斯拉FSD入华...</td>
    <td style="padding:6px; white-space:nowrap;">特斯</td>
    <td style="padding:6px;">4.1k</td>
    <td style="padding:6px; color:#67C23A;">😃60%</td>
  </tr>
</table>
`;

const messages = ref<Message[]>([
  {
    role: "assistant",
    content: "您好！我是VOC智能助手，有什么可以帮您?",
    time: getCurrentTime(),
  },
  {
    role: "assistant",
    content: competitorTableHtml,
    time: getCurrentTime(),
  }
]);

const handleMessageClick = (e: MouseEvent) => {
  const target = (e.target as HTMLElement).closest('[data-action]');
  if (!target) return;
  
  const action = target.getAttribute('data-action');
  const topic = target.getAttribute('data-topic');
  
  if (action === 'navigate-competitor' && topic) {
    isOpen.value = false; // 关闭聊天窗
    if (navigate) {
      navigate('competitor', { topic });
    }
  }
};

const toggleChat = () => {
  isOpen.value = !isOpen.value;
  if (isOpen.value) {
    unreadCount.value = 0;
    nextTick(() => {
      scrollToBottom();
    });
  }
};

const startNewChat = () => {
  messages.value = [
    {
      role: "assistant",
      content: `您好！我是VOC智能助手，有什么可以帮您?`,
      time: new Date().toLocaleTimeString("zh-CN", {
        hour: "2-digit",
        minute: "2-digit",
      }),
    },
  ];
  inputText.value = "";
  nextTick(() => {
    scrollToBottom();
  });
};

const handleSend = () => {
  if (!inputText.value.trim() || isTyping.value) return;
  sendMessage(inputText.value);
  inputText.value = "";
};

const sendMessage = (text: string) => {
  // 添加用户消息
  messages.value.push({
    role: "user",
    content: text,
    time: getCurrentTime(),
  });

  scrollToBottom();

  // 模拟AI回复
  isTyping.value = true;

  setTimeout(() => {
    isTyping.value = false;

    // 根据问题生成对应回复
    let reply = "";
    if (text.includes("摄像头")) {
      reply =
        "H56D配备了<strong>12个摄像头</strong>，包括：<br/>• 前视摄像头：3个<br/>• 侧视摄像头：4个<br/>• 后视摄像头：4个<br/>• 车内摄像头：1个";
    } else if (text.includes("重置") || text.includes("车机系统")) {
      reply =
        "车机系统重置步骤如下：<br/>1. 进入<strong>设置</strong> → <strong>系统</strong><br/>2. 选择<strong>恢复出厂设置</strong><br/>3. 输入密码确认<br/>4. 等待系统自动重启<br/><br/>⚠️ 重置前请备份重要数据！";
    } else if (text.includes("充电枪") || text.includes("拔不下来")) {
      reply =
        "充电枪拔不下来的解决方法：<br/>1. <strong>先尝试解锁车辆</strong>（按遥控器解锁键）<br/>2. <strong>检查充电状态</strong>：确保充电已完全停止<br/>3. <strong>手动释放</strong>：打开充电口盖板，找到应急释放拉环<br/>4. 如仍无法解决，请联系<strong>4S店</strong>或<strong>道路救援</strong>";
    } else {
      reply = `关于"${text}"的问题，我已收到。基于VOC数据分析，建议您查看相关功能模块获取详细信息。`;
    }

    messages.value.push({
      role: "assistant",
      content: reply,
      time: getCurrentTime(),
    });

    scrollToBottom();
  }, 1500);
};

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
  });
};

function getCurrentTime() {
  const now = new Date();
  return `${now.getHours().toString().padStart(2, "0")}:${now.getMinutes().toString().padStart(2, "0")}`;
}
</script>

<style scoped>
.chatbot-wrapper {
  position: fixed;
  right: 30px;
  bottom: 30px;
  z-index: 9999;
}

.chatbot-trigger {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
  transition: all 0.3s;
}

.chatbot-trigger:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
}

.chatbot-dialog {
  position: fixed;
  right: 30px;
  bottom: 100px;
  width: 380px;
  height: 600px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}


.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(20px);
  opacity: 0;
}

.chat-header {
  padding: 16px 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.header-text {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.header-title {
  font-size: 16px;
  font-weight: 600;
}

.header-status {
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 6px;
  opacity: 0.9;
}

.status-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #67c23a;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.close-btn {
  cursor: pointer;
  font-size: 20px;
  padding: 4px;
  border-radius: 4px;
  transition: background 0.3s;
}

.close-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.new-chat-btn {
  cursor: pointer;
  font-size: 18px;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.3s;
}

.new-chat-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: rotate(90deg);
}

.chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  background: #f5f7fa;
}

/* 滚动条美化 */
.chat-messages::-webkit-scrollbar {
  width: 6px;
}

.chat-messages::-webkit-scrollbar-track {
  background: transparent;
}

.chat-messages::-webkit-scrollbar-thumb {
  background-color: #cdbcf3; /* 淡紫色 */
  border-radius: 3px;
}

.chat-messages::-webkit-scrollbar-thumb:hover {
  background-color: #b19af0; /* 深一点的紫色 */
}

.message {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message.assistant {
  justify-content: flex-start;
}

.message.user {
  justify-content: flex-end;
}

.message-content {
  max-width: 88%;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.6;
}

.message.assistant .message-bubble {
  background: #fff;
  color: #303133;
  border-bottom-left-radius: 4px;
}

.message.user .message-bubble {
  background: #409eff;
  color: #fff;
  border-bottom-right-radius: 4px;
}

.message-time {
  font-size: 12px;
  color: #909399;
  margin-top: 4px;
  text-align: right;
}

.message.assistant .message-time {
  text-align: left;
}

.typing-indicator {
  background: #fff;
  padding: 12px 16px;
  border-radius: 12px;
  display: flex;
  gap: 4px;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #909399;
  animation: typing 1.4s infinite;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing {
  0%,
  60%,
  100% {
    transform: translateY(0);
  }
  30% {
    transform: translateY(-10px);
  }
}

.quick-questions {
  padding: 0 20px 10px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  background: #f5f7fa;
}

.quick-question {
  padding: 10px 14px;
  background: #fff;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.3s;
  color: #606266;
}

.quick-question:hover {
  border-color: #409eff;
  color: #409eff;
  transform: translateX(4px);
}

.chat-input {
  padding: 16px 20px;
  background: #fff;
  border-top: 1px solid #dcdfe6;
}

.chat-input :deep(.el-input__wrapper) {
  border-radius: 20px;
}
</style>
