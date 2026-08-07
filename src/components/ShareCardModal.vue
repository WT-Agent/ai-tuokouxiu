<template>
  <div v-if="visible" class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content share-modal-content">
      <div class="modal-header">
        <h3>分享卡片生成</h3>
        <button class="close-icon-btn" @click="$emit('close')">×</button>
      </div>

      <!-- 玻璃拟态分享卡片实体区域 (可通过 H5 长按或截图保存) -->
      <div id="share-card-node" class="share-card-wrapper">
        <div class="share-card-header">
          <div class="share-brand">
            <span class="brand-logo-dot"></span>
            <span class="brand-title">网腾无限AI</span>
          </div>
          <span class="share-tag">爆款文案二创</span>
        </div>

        <div class="share-card-body">
          <div class="share-quote-symbol">“</div>
          <div class="share-result-text">{{ content }}</div>
          <div class="share-quote-symbol end">”</div>
        </div>

        <div class="share-card-footer">
          <div class="share-qrcode-area">
            <div class="qr-placeholder">
              <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <rect x="3" y="3" width="7" height="7" rx="1"></rect>
                <rect x="14" y="3" width="7" height="7" rx="1"></rect>
                <rect x="3" y="14" width="7" height="7" rx="1"></rect>
                <path d="M14 14h3v3h-3z"></path>
                <path d="M18 18h3v3h-3z"></path>
              </svg>
            </div>
            <div class="share-tip-text">
              <span class="primary">扫码解锁无限创作灵感</span>
              <span class="sub">ai.wuxian.xyz · 微信公众号：{{ wechatId }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 操作与提示 -->
      <div class="share-actions">
        <p class="share-guide-tip">微信 H5 环境下可点击下方按钮复制，或直接截图保存此精美卡片到相册</p>
        <div class="share-btn-group">
          <button class="modal-btn secondary" @click="$emit('close')">关闭</button>
          <button class="modal-btn primary" @click="handleCopy">
            {{ copied ? '卡片文案已复制' : '复制卡片全文本' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const props = defineProps<{
  visible: boolean;
  content: string;
  wechatId: string;
}>();

defineEmits<{
  (e: 'close'): void;
}>();

const copied = ref(false);

const handleCopy = async () => {
  try {
    const textToCopy = `【网腾无限AI - 爆款文案二创】\n\n${props.content}\n\n体验更多 AI 灵感微应用：https://ai.wuxian.xyz`;
    await navigator.clipboard.writeText(textToCopy);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    console.error('复制失败', err);
  }
};
</script>
