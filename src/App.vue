<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div v-if="copied" class="top-success-toast">
      复制成功
    </div>
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 实战引擎 · 解决高效生产力需求</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <div class="selector-group">
        <label class="selector-label">输入您要生成的内容或要求</label>
        <textarea 
          v-model="userInput" 
          placeholder="比如：帮我写一段表达工作辛苦但充满希望的总结..."
        ></textarea>
      </div>

      <div class="selector-group">
        <label class="selector-label">选择生成风格</label>
        <div class="style-selector">
          <button 
            v-for="style in styleOptions" 
            :key="style.value"
            class="style-option"
            :class="{ active: activeStyle === style.value }"
            @click="activeStyle = style.value"
          >
            {{ style.label }}
          </button>
        </div>
      </div>

      <button 
        class="action-btn" 
        :disabled="loading || !userInput.trim()"
        @click="handleGenerate"
      >
        {{ loading ? '正在飞速生成中...' : '开始一键生成' }}
      </button>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">生成结果</span>
        <div class="button-actions">
          <button v-if="result && !isImageProject" class="icon-btn" @click="copyText">
            {{ copied ? '已复制' : '复制文案' }}
          </button>
          <a v-if="result && isImageProject" :href="result" target="_blank" download class="icon-btn" style="text-decoration: none;">
            查看原图
          </a>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 80%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <img v-if="isImageProject" :src="result" alt="Generated visual" class="image-output" />
        <div v-else class="output-content">{{ result }}</div>
      </div>
    </section>

    <!-- PC端 Nomads 案例与模版展示 -->
    <NomadsShowcase
      :app-title="appTitle"
      :is-image="isImageProject"
      @apply-template="handleApplyTemplate"
    />

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的所有文本或图像提示词仅用于实时大模型生成，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的免费额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 微应用服务。使用本应用即代表您同意并承诺遵守当地有关人工智能生成内容（AIGC）的法律法规。</p>
          <p>所有生成结果均由 AI 模型计算产生，本应用不对生成内容的准确性、完整性及合法性承担任何直接或间接法律责任。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

<!-- 联系我们弹窗 -->
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

const CDN_ASSET_BASE = 'https://ai.wuxian.xyz/assets';
const weixinImg = `${CDN_ASSET_BASE}/weixin.png`;
const dingtalkImg = `${CDN_ASSET_BASE}/dingtalk.png`;

onMounted(() => {
  const savedTheme = localStorage.getItem('portal_theme') || 'dark';
  document.documentElement.setAttribute('data-theme', savedTheme);
});

// 读取动态配置文件配置
const appTitle = ref(appConfig.title || 'AI微应用');
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

// 解析 Cookie 辅助函数
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

// SSO 用户状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 判断当前项目是文本类还是图像/多模态类
const isImageProject = computed(() => {
  return appConfig.type === 'image';
});

// 根据生成类别提供不同的风格预设
const styleOptions = computed(() => {
  if (isImageProject.value) {
    return [
      { label: '写真照片', value: '<photography>' },
      { label: '卡通动漫', value: '<anime>' },
      { label: '水彩画卷', value: '<watercolor>' },
      { label: '插画艺术', value: '<illustration>' },
    ];
  } else {
    return [
      { label: '专业干练', value: '专业干练，结果导向' },
      { label: '高情商说辞', value: '高情商，委婉，有情调' },
      { label: '幽默风趣', value: '幽默风趣，形象生动' },
      { label: '严谨学术', value: '严谨学术，条理清晰' },
    ];
  }
});

const activeStyle = ref(styleOptions.value[0].value);

// 判断是否达到免费次数上限
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

// 获取 API 请求端点
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

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: isImageProject.value ? 'image' : 'text',
        prompt: `类型：${promptTopic.value}，要求：${userInput.value}，风格倾向：${activeStyle.value}`,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      
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

const handleApplyTemplate = (payload: { prompt: string; style?: string }) => {
  userInput.value = payload.prompt;
  if (payload.style) {
    activeStyle.value = payload.style;
  }
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(result.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
