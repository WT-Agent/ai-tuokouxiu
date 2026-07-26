<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  {
    "text": "用户 138****9911 完成了开放麦 3 分钟职场加班脱口秀逐字稿"
  },
  {
    "text": "用户 189****4422 得到了 3 个关于相亲奇葩经历的爆笑 Punchline"
  },
  {
    "text": "用户 131****7700 完成了减肥吃草自嘲段子与表演节奏标注"
  },
  {
    "text": "用户 156****3355 生成了针对现场互动观众的机智 CrowdWork 救场词"
  },
  {
    "text": "用户 173****8899 取得了脱口秀谐音梗与比喻梗创作 SOP"
  }
]);
</script>
