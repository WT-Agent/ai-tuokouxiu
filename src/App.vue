<template>
  <div class="app-container">
    <!-- 常驻悬浮分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享应用</span>
    </button>

    <header>
      <div class="user-status-bar" style="margin-bottom: 0.75rem; font-size: 0.8rem; text-align: center;">
        <span v-if="isLoggedIn" class="status-badge logged-in" style="background: rgba(192, 132, 252, 0.15); color: #c084fc; padding: 4px 12px; border-radius: 12px; border: 1px solid rgba(192, 132, 252, 0.3);">
          已登录 (每日 15 次额度 · 今日已用: {{ authUsesCount }}/15)
        </span>
      </div>
      <h1>{{ appTitle }}</h1>
      <p>专业生成 · 智能诊断 · 5大指标评估 · 极简自适应体验</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 预设分类选择 -->
      <div class="selector-group">
        <label class="selector-label">选择场景与需求类型</label>
        <div class="style-selector">
          <button 
            v-for="ptype in presetOptions" 
            :key="ptype"
            class="style-option"
            :class="{ active: activePresetType === ptype }"
            @click="activePresetType = ptype"
          >
            {{ ptype }}
          </button>
        </div>
      </div>

      <!-- 属性选择行 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">吐槽主题</label>
          <div class="style-selector">
            <button 
              v-for="attr in attr1Options" 
              :key="attr"
              class="style-option"
              :class="{ active: selectedAttr1 === attr }"
              @click="selectedAttr1 = attr"
            >
              {{ attr }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">喜剧风格</label>
          <div class="style-selector">
            <button 
              v-for="attr in attr2Options" 
              :key="attr"
              class="style-option"
              :class="{ active: selectedAttr2 === attr }"
              @click="selectedAttr2 = attr"
            >
              {{ attr }}
            </button>
          </div>
        </div>
      </div>

      <!-- 用户输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入您的详细内容、场景背景与要点需求</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showGuideModal = true">脱口秀段子创作与舞台演绎通关指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请简要描述您的需求背景与要点...（内容越详细，生成的报告越精准专业）"
          style="min-height: 130px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议详细补充背景细节、目标诉求与关注要点</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在分析与专业生成中...' : '开始生成专业报告' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">专业生成指导报告</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制全文' : '复制报告' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 共识打分可视化看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 专业维度评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合得分: {{ getAverageScoreFromMap(aiScores) }} / 5.0</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item">
              <div style="display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 0.3rem;">
                <span style="color: var(--text-secondary);">{{ metric.label }}</span>
                <span style="font-weight: bold; color: var(--accent-color);">{{ aiScores[metric.key] || 4 }} / 5</span>
              </div>
              <div class="bar-bg" style="height: 6px; background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;">
                <div class="bar-fill" :style="{ width: ((aiScores[metric.key] || 4) * 20) + '%', background: 'var(--primary-gradient)', height: '100%', borderRadius: '3px', transition: 'width 0.5s ease' }"></div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- 历史记录面板 -->
    <section v-if="showHistory" class="glass-card" style="margin-top: 1rem;">
      <div class="result-header">
        <span class="result-title">本地生成历史记录</span>
        <button class="icon-btn" @click="showHistory = false">关闭记录</button>
      </div>

      <div v-if="historyList.length === 0" style="text-align: center; color: var(--text-secondary); padding: 1.5rem; font-size: 0.85rem;">
        暂无历史生成记录，开始体验吧！
      </div>

      <div v-else class="history-grid" style="display: flex; flex-direction: column; gap: 0.75rem; max-height: 320px; overflow-y: auto;">
        <div v-for="item in historyList" :key="item.id" class="history-item" style="padding: 1rem; background: rgba(0,0,0,0.2); border-radius: 10px; border: 1px solid var(--card-border);">
          <div style="display: flex; justify-content: space-between; font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
            <span>{{ item.timestamp }} · [{{ item.presetType }}]</span>
            <span style="color: var(--primary-color);">评分: {{ getAverageScore(item) }}</span>
          </div>
          <div style="font-size: 0.85rem; font-weight: bold; margin-bottom: 0.4rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: var(--text-primary);">
            内容: {{ item.input }}
          </div>
          <div style="display: flex; gap: 0.5rem;">
            <button class="icon-btn" style="font-size: 0.75rem;" @click="applyHistory(item)">套用内容</button>
            <button class="icon-btn" style="font-size: 0.75rem;" @click="viewHistoryOutput(item)">查看全文</button>
          </div>
        </div>
      </div>
    </section>

    <!-- 场景模版 Showcase -->
    <NomadsShowcase @apply-template="handleApplyTemplate" />

    <!-- 领域指南 Modal -->
    <div v-if="showGuideModal" class="modal-overlay" @click.self="showGuideModal = false">
      <div class="modal-content" style="max-width: 480px;">
        <h3>脱口秀段子创作与舞台演绎通关指南</h3>
        <p style="text-align: left; font-size: 0.825rem; margin-bottom: 1rem; color: var(--text-secondary);">
          专业指导与关键避坑法则：
        </p>
        <div class="modal-scroll-area" style="text-align: left; font-size: 0.825rem;">
          <div v-for="(rule, idx) in guideRules" :key="idx" style="margin-bottom: 0.75rem; padding: 0.5rem 0.75rem; background: rgba(255,255,255,0.03); border-radius: 8px; border: 1px solid rgba(255,255,255,0.05);">
            <div style="color: var(--accent-color); font-weight: bold; margin-bottom: 0.2rem;">{{ rule.title }}</div>
            <div style="color: var(--text-primary); margin-bottom: 0.2rem;">正向建议: {{ rule.advice }}</div>
            <div style="color: var(--text-secondary); font-size: 0.775rem;">避坑避雷: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showGuideModal = false">关闭</button>
      </div>
    </div>

    <!-- 微信 H5 分享引导 Modal -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享应用</h3>
        <p>扫码关注或将链接转发给需要的朋友，随时体验专业 AI 服务。</p>
        
        <div class="qr-code-placeholder">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%">
            <rect width="100" height="100" fill="white"/>
            <rect x="5" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="9" width="17" height="17" fill="white"/>
            <rect x="13" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="70" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="74" y="9" width="17" height="17" fill="white"/>
            <rect x="78" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="5" y="70" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="74" width="17" height="17" fill="white"/>
            <rect x="13" y="78" width="9" height="9" fill="#110e24"/>
            <rect x="35" y="10" width="8" height="8" fill="#110e24"/>
            <rect x="48" y="5" width="6" height="12" fill="#110e24"/>
            <rect x="60" y="15" width="5" height="5" fill="#110e24"/>
            <rect x="35" y="35" width="10" height="10" fill="#110e24"/>
            <rect x="50" y="45" width="15" height="8" fill="#110e24"/>
            <rect x="40" y="70" width="8" height="16" fill="#110e24"/>
            <rect x="55" y="65" width="10" height="10" fill="#110e24"/>
            <rect x="75" y="40" width="12" height="12" fill="#110e24"/>
            <rect x="75" y="75" width="15" height="15" fill="#110e24"/>
            <rect x="45" y="80" width="8" height="8" fill="#110e24"/>
          </svg>
        </div>

        <div style="font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 1.5rem;">
          微信号: <span style="color: var(--primary-color); font-weight: bold;">{{ wechatId }}</span>
        </div>

        <button class="modal-btn" @click="showShareGuide = false">关闭</button>
      </div>
    </div>

    <!-- 底部条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 弹窗组件 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们高度重视您的个人隐私与数据安全。您在本应用中输入的内容仅用于大模型实时分析生成，不会在云端公开。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用网腾无限 AI 智能服务。本工具生成的内容供参考与辅助决策。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI - 脱口秀段子与爆笑金句专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);

const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showGuideModal = ref(false);

const getCookie = (name: string): string | null => {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

const presetOptions = [
  "打工人吐槽 PPT 汇报与开会文化的 3 分钟脱口秀",
  "当代单身青年被父母疯狂催婚与相亲尴尬段子",
  "减肥刷脂期的痛苦与自律伪精致喜剧逐字稿",
  "00 后整顿职场与代沟冲突开放麦段子"
];
const activePresetType = ref(presetOptions[0]);

const attr1Options = [
  "职场/开会/画饼",
  "催婚/相亲/恋爱",
  "减肥/健身/消费",
  "代沟/父母/童年"
];
const selectedAttr1 = ref(attr1Options[0]);

const attr2Options = [
  "冷面笑匠冷嘲风",
  "夸张肢体 Act-out 风",
  "逻辑思辨解构风",
  "互动砸缸控场风"
];
const selectedAttr2 = ref(attr2Options[0]);

const metricsList = [
  {
    "key": "punchlineLaughterExplosion",
    "label": "Punchline 爆笑指数"
  },
  {
    "key": "setupBuildUpLogic",
    "label": "铺垫逻辑铺设度"
  },
  {
    "key": "selfDeprecationHumor",
    "label": "自嘲幽默分寸感"
  },
  {
    "key": "stageDeliveryRhythm",
    "label": "现场演绎卡拍节奏"
  },
  {
    "key": "viralMemePotential",
    "label": "爆款造梗传播力"
  }
];
const guideRules = [
  {
    "title": "Setup 铺垫要真实具体",
    "advice": "铺垫越真实详细（如“昨天下午 5 点半老板突然发消息”），后半句 Punchline 的反转威力越大",
    "avoid": "切忌铺垫过于空洞导致观众无法建立预期"
  },
  {
    "title": "自嘲是最好的保护色",
    "advice": "多拿自己的尴尬、失败与缺点开涮（自嘲），更容易赢得观众的善意与笑声",
    "avoid": "避免居高临下指责观众或过度攻击特定群体"
  },
  {
    "title": "表演 Act-out 加分",
    "advice": "文字不够肢体来凑，在段子高潮处模仿反派语气或夸张动作（Act-out），能瞬间引爆全场",
    "avoid": "切勿干巴巴背诵文字稿而没有语调起伏"
  }
];

const aiScores = ref<Record<string, number> | null>(null);

interface HistoryItem {
  id: string;
  timestamp: string;
  presetType: string;
  input: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[TUOKOUXIU_SCORES\][\s\S]*?\[\/TUOKOUXIU_SCORES\]/g, '').trim();
});

const parseAiScores = (rawText: string) => {
  const match = rawText.match(/\[TUOKOUXIU_SCORES\](.*?)\[\/TUOKOUXIU_SCORES\]/);
  if (!match) return null;
  const content = match[1];
  const scoresObj: Record<string, number> = {};
  content.split(',').forEach(i => {
    const [key, val] = i.split(':');
    if (key && val) {
      scoresObj[key.trim()] = parseInt(val.trim(), 10) || 4;
    }
  });
  return Object.keys(scoresObj).length > 0 ? scoresObj : null;
};

const getAverageScoreFromMap = (scores: Record<string, number>) => {
  const keys = Object.keys(scores);
  if (keys.length === 0) return '4.5';
  const sum = keys.reduce((acc, k) => acc + (scores[k] || 4), 0);
  return (sum / keys.length).toFixed(1);
};

const getAverageScore = (historyItem: HistoryItem) => {
  if (!historyItem.aiScores) return '4.5';
  return getAverageScoreFromMap(historyItem.aiScores);
};

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('tuokouxiu_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('tuokouxiu_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    presetType: activePresetType.value,
    input: userInput.value,
    aiScores: aiScores.value,
    output: result.value
  };
  historyList.value.unshift(newItem);
  if (historyList.value.length > 20) {
    historyList.value = historyList.value.slice(0, 20);
  }
  saveHistory();
};

const toggleHistoryDrawer = () => {
  loadHistory();
  showHistory.value = !showHistory.value;
};

const applyHistory = (historyItem: HistoryItem) => {
  userInput.value = historyItem.input;
  activePresetType.value = historyItem.presetType;
  showHistory.value = false;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (historyItem: HistoryItem) => {
  userInput.value = historyItem.input;
  result.value = historyItem.output;
  aiScores.value = historyItem.aiScores;
  showHistory.value = false;
};

const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: `任务指导: ${promptTopic.value}\n【分类场景】: ${activePresetType.value}\n【${'吐槽主题'}】: ${selectedAttr1.value}\n【${'喜剧风格'}】: ${selectedAttr2.value}\n【详细输入】: ${userInput.value}`,
        style: activePresetType.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAiScores(data.result);
      
      addHistoryRecord();

      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: { prompt: string }) => {
  userInput.value = payload.prompt;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const resetResult = () => {
  result.value = '';
  aiScores.value = null;
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};

onMounted(() => {
  loadHistory();
});
</script>

<style scoped>
.text-link-btn {
  background: none;
  border: none;
  color: #a5b4fc;
  font-size: 0.775rem;
  cursor: pointer;
  transition: color 0.2s ease;
}
.text-link-btn:hover {
  color: var(--text-primary);
  text-decoration: underline;
}
</style>
