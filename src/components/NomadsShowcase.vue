<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">实战案例与模板库 (Nomads Showcase)</h2>
        <p class="showcase-subtitle">精选高频实战场景，点击“一键套用”快速生成高质量结果</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个实战模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次应用</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  appTitle?: string;
  isImage?: boolean;
}>();

const emit = defineEmits<{
  (e: 'apply-template', payload: { prompt: string; style?: string }): void;
}>();

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  style?: string;
  usageCount: string;
}

// 模拟实战案例数据库（支持根据文本/图像类及应用主题切换）
const showcaseItems = computed<ShowcaseItem[]>(() => {
  if (props.isImage) {
    return [
      {
        id: 'img-1',
        tag: '写真肖像',
        title: '商务精英形象照',
        prompt: '高端写字楼背景，身穿深蓝色西装，眼神自信专注，赛博朋克光影效果',
        style: '<photography>',
        usageCount: '18.5k'
      },
      {
        id: 'img-2',
        tag: '概念插画',
        title: '未来科幻城市海报',
        prompt: '霓虹灯光的赛博朋克立体城市，飞行汽车，高品质概念插画，8k分辨率',
        style: '<illustration>',
        usageCount: '24.1k'
      },
      {
        id: 'img-3',
        tag: '二次元动漫',
        title: '日系国潮动漫角色',
        prompt: '穿着现代汉服的国风少年，手持纸伞，水彩漫感，唯美光感与柔光滤镜',
        style: '<anime>',
        usageCount: '15.9k'
      },
      {
        id: 'img-4',
        tag: '水彩艺术',
        title: '治愈系自然风景画',
        prompt: '晨雾中的森林湖泊，阳光穿透树林，水彩渐变质感，温馨治愈风格',
        style: '<watercolor>',
        usageCount: '12.3k'
      }
    ];
  } else {
    return [
      {
        id: 'text-1',
        tag: '职场总结',
        title: '周报 OKR 成果提炼',
        prompt: '本周完成了核心模块优化与线上异常排查，请帮我梳理为具备量化指标的 OKR 汇报文案',
        style: '专业干练，结果导向',
        usageCount: '32.8k'
      },
      {
        id: 'text-2',
        tag: '高情商沟通',
        title: '拒绝不合理加班话术',
        prompt: '领导在周末突然布置非紧急任务，如何高情商、委婉且有理有据地推迟到工作日处理？',
        style: '高情商，委婉，有情调',
        usageCount: '28.4k'
      },
      {
        id: 'text-3',
        tag: '短视频文案',
        title: '知识干货吸睛开头',
        prompt: '准备制作一条关于高效学习法的短视频，设计 3 个能在前 3 秒留住用户的爆款口播开头',
        style: '专业干练，结果导向',
        usageCount: '45.1k'
      },
      {
        id: 'text-4',
        tag: '商务公文',
        title: '跨部门协同申请函',
        prompt: '因项目上线需要研发部门配合联调，撰写一份正式、严谨且明确时间节点的协同申请书',
        style: '专业干练，结果导向',
        usageCount: '19.7k'
      },
      {
        id: 'text-5',
        tag: '小红书种草',
        title: '实战干货笔记排版',
        prompt: '分享 5 个提升日常工作效率的文字工具，语言亲切，搭配吸引人的标题与 Emoji 排版',
        style: '高情商，委婉，有情调',
        usageCount: '36.2k'
      },
      {
        id: 'text-6',
        tag: '学术润色',
        title: '论文摘要与结论重构',
        prompt: '将以下粗糙的研究结论重写为学术规范、逻辑严密且无语法语病的论文摘要总结',
        style: '专业干练，结果导向',
        usageCount: '22.0k'
      }
    ];
  }
});

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    style: item.style
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
