# Follow Builder — AI Builders Morning Report

> **改进行声明：** 本技能基于 Zara Zhang 的 [Follow Builders](https://github.com/zarazhangrui/follow-builders) 技能进行深度改进。原始技能专注于追踪 AI 领域真正的建设者（Builders），本改进版在保留其核心数据管线的基础上，新增**橙红渐变晨报风 HTML 输出**，提供更丰富的视觉呈现和阅读体验。

---

## 核心哲学

"Follow Builders, Not Influencers" —— 追踪真正在做产品、开公司、做研究的 AI 建设者，而非人云亦云的意见领袖。

**数据管线（继承自原版）：** 所有 X/Twitter 推文、播客字幕、博客文章均由中心 feed 自动抓取，用户无需提供任何 API key。本技能仅负责将 JSON 内容重新编排为 HTML 晨报。

---

## 改进亮点

| # | 改进项 | 说明 |
|---|--------|------|
| 1 | **HTML 晨报输出** | 纯 HTML/CSS/JS 单文件，橙红渐变 Hero，响应式卡片网格 |
| 2 | **编号贯穿全文** | 全局连续序号，一目了然今日总条数 |
| 3 | **五大内容版块** | 模型与前沿、产品与发布、商业与投资、工程与工具、深度播客 |
| 4 | **锚点导航** | 顶部固定导航栏，一键跳转到各版块 |
| 5 | **北京人话时间** | "今天上午 09:48" / "2 小时前" / "昨天下午 15:30" |
| 6 | **滚动渐入动画** | Intersection Observer 驱动，卡片逐个淡入 |
| 7 | **系统字体栈** | CSS 变量驱动，无外部依赖 |
| 8 | **本地 HTML 文件** | 保存至工作区，可在浏览器直接打开 |

---

## 平台检测

首先检测运行平台：

```bash
which openclaw 2>/dev/null && echo "PLATFORM=openclaw" || echo "PLATFORM=other"
```

- **OpenClaw** (`PLATFORM=openclaw`): 持久 Agent，有内置消息通道。使用 `openclaw cron add` 设置定时任务。
- **Other** (Claude Code, Cursor 等): 非持久 Agent。用户键入 `/ai` 或对应触发词获取晨报。

---

## 首次运行 — 引导流程

检查 `~/.follow-builders/config.json` 是否存在且 `onboardingComplete: true`。若否，运行引导流程：

### Step 1: 介绍

告诉用户：

"我是你的 AI Builders 晨报助手。我追踪 AI 领域真正的建设者——研究员、创始人、PM、工程师——他们在 X/Twitter 和 YouTube 播客上分享的内容。每天我会为你生成一份橙红渐变晨报风的 HTML 日报。"

### Step 2: 发送偏好

询问：
- "你希望晨报频率是？" → 每日 / 每周
- "几点发送？什么时区？" → 例如 "早上 8 点，北京时间"

### Step 3: 发送方式

**若 OpenClaw:** 跳过此步。设置 `delivery.method` 为 `"stdout"`。

**若非持久 Agent:**
告知用户可通过 Telegram / Email 自动发送，或按需手动触发（键入 `/ai`）。按原版流程引导。

### Step 4: 语言

"你希望晨报使用什么语言？"
- 简体中文（推荐）
- 中英双语

### Step 5-8: API Key、源列表、配置提醒、Cron 设置

沿用原版 follow-builders 流程，保持不变。

### Step 9: 欢迎晨报

**必须执行。** 立即生成第一份 HTML 晨报，让用户预览效果。

---

## 内容生成 — 每日晨报流程

### Step 1: 加载配置

读取 `~/.follow-builders/config.json` 获取用户偏好。

### Step 2: 运行数据准备脚本

```bash
cd /Users/charliedelta/.workbuddy/skills/followbuilder/scripts && node prepare-digest.js 2>/dev/null
```

脚本输出 JSON blob，包含：config、podcasts、x、blogs、stats、prompts、errors。

**ABSOLUTE RULES:**
- **NEVER** 自己访问 x.com、搜索网页或调用任何 API
- **NEVER** 编造内容。只能使用 JSON 中已有的数据
- 每条内容必须有 URL。没有 URL = 不放入晨报

### Step 3: 检查是否有内容

若 `stats.podcastEpisodes` 为 0 且 `stats.xBuilders` 为 0，告知用户 "今天没有新的建设者动态，明天见！"

### Step 4: 内容编排（Remix）

读取 JSON 中 `prompts` 字段的提示词，按以下规则编排：

**推文处理（优先）：**
- `x` 数组中每个 builder 带有 tweets 数组
- 使用 `bio` 字段确定角色（如 "Box CEO Aaron Levie"）
- 按 `prompts.summarize_tweets` 摘要
- 每条推文必须包含其 `url`

**播客处理：**
- `podcasts` 数组最多 1 个 episode
- 使用 `prompts.summarize_podcast` 摘要
- 包含 `name`、`title`、`url`

**博客处理：**
- `blogs` 数组中的文章
- 使用 `prompts.summarize_blogs` 摘要

**内容分类到五大版块：**
将每条内容分配到以下五个版块之一：

| 版块 | 英文名 | 覆盖内容 |
|------|--------|---------|
| 🤖 模型与前沿 | Models & Research | 模型发布、论文解读、基准测试、前沿突破 |
| 🚀 产品与发布 | Products & Launches | 产品发布、功能更新、新工具、SDK |
| 💼 商业与投资 | Business & Investment | 融资、收购、战略调整、行业趋势 |
| 🔧 工程与工具 | Engineering & Tools | 开发实践、基础设施、开源项目、架构 |
| 🎙️ 深度播客 | Podcast Deep Dives | 播客长内容、深度访谈、技术长文 |

### Step 5: 生成 HTML 晨报

基于编排后的内容，按照下方的 **HTML 晨报模板规范** 生成完整的 HTML 文件。

**关键要求：**
1. 所有内容均包含编号（从 1 开始全局递增）
2. Hero 区域展示日期、总条数、五大版块统计
3. 每条卡片包含：全局序号、标题、来源 chip（X/Podcast/Blog）、详细内容、原文跳转按钮
4. 链接使用 `target="_blank" rel="noopener noreferrer"`
5. 时间转换为北京时间人话格式
6. 文末标注总条数和数据源
7. 纯 HTML/CSS/JS 单文件，零外部依赖

### Step 6: 保存并呈现

将生成的 HTML 保存到工作区：

```bash
DATE=$(date +%Y-%m-%d)
mkdir -p /Users/charliedelta/WorkBuddy/2026-06-29-09-46-47/outputs
```

写入文件：`/Users/charliedelta/WorkBuddy/2026-06-29-09-46-47/outputs/ai-builders-${DATE}.html`

然后调用 `present_files` 向用户展示。

---

## HTML 晨报模板规范

### 设计系统 — CSS 变量

```css
:root {
  /* 橙红渐变主色调 */
  --hero-gradient: linear-gradient(135deg, #FF6B35 0%, #E8453A 40%, #D32F2F 70%, #B71C1C 100%);
  --hero-gradient-light: linear-gradient(135deg, #FF8A65 0%, #EF5350 40%, #E53935 70%, #D32F2F 100%);

  /* 背景色 */
  --bg-primary: #FFF5F0;
  --bg-secondary: #FFFFFF;
  --bg-card: #FFFFFF;
  --bg-card-hover: #FFF8F5;

  /* 文字色 */
  --text-primary: #2D1B15;
  --text-secondary: #6B4A40;
  --text-muted: #9E7A6B;
  --text-hero: #FFFFFF;

  /* 强调色 */
  --accent-orange: #FF6B35;
  --accent-red: #E8453A;
  --accent-gold: #FFB74D;

  /* 版块色 */
  --section-models: #FF6B35;
  --section-products: #E8453A;
  --section-business: #D32F2F;
  --section-engineering: #FF8A65;
  --section-podcast: #B71C1C;

  /* 卡片 */
  --card-shadow: 0 2px 12px rgba(180, 60, 40, 0.08);
  --card-shadow-hover: 0 6px 24px rgba(180, 60, 40, 0.15);
  --card-radius: 16px;
  --card-border: 1px solid rgba(255, 107, 53, 0.12);

  /* 排版 */
  --font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans SC", "PingFang SC", "Microsoft YaHei", sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.5rem;
  --font-size-2xl: 2rem;
  --font-size-hero: 2.5rem;

  /* 间距 */
  --spacing-xs: 0.5rem;
  --spacing-sm: 0.75rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  --spacing-2xl: 3rem;

  /* 动画 */
  --transition-fast: 0.2s ease;
  --transition-normal: 0.3s ease;
  --transition-slow: 0.5s ease;
}
```

### HTML 结构

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="color-scheme" content="light">
  <title>AI Builders 晨报 — 2026年6月29日</title>
  <style>/* 全部 CSS 内联 */</style>
</head>
<body>
  <!-- ===== 1. HERO 区域 ===== -->
  <header class="hero">
    <div class="hero-content">
      <h1 class="hero-title">AI Builders 晨报</h1>
      <div class="hero-date">2026年6月29日 星期一</div>
      <div class="hero-count">今日共 <strong>N</strong> 条资讯</div>
      <div class="hero-stats">
        <span class="stat-item">🤖 模型与前沿: X条</span>
        <span class="stat-item">🚀 产品与发布: X条</span>
        <span class="stat-item">💼 商业与投资: X条</span>
        <span class="stat-item">🔧 工程与工具: X条</span>
        <span class="stat-item">🎙️ 深度播客: X条</span>
      </div>
    </div>
  </header>

  <!-- ===== 2. 锚点导航 ===== -->
  <nav class="anchor-nav" id="anchorNav">
    <a href="#section-models">🤖 模型与前沿</a>
    <a href="#section-products">🚀 产品与发布</a>
    <a href="#section-business">💼 商业与投资</a>
    <a href="#section-engineering">🔧 工程与工具</a>
    <a href="#section-podcast">🎙️ 深度播客</a>
  </nav>

  <!-- ===== 3. 内容区 ===== -->
  <main class="content">
    <!-- 每个版块一个 section -->
    <section id="section-models" class="section-block">...</section>
    <section id="section-products" class="section-block">...</section>
    <section id="section-business" class="section-block">...</section>
    <section id="section-engineering" class="section-block">...</section>
    <section id="section-podcast" class="section-block">...</section>
  </main>

  <!-- ===== 4. 页脚 ===== -->
  <footer class="footer">
    共 N 条资讯 | 数据来源: X/Twitter, YouTube Podcasts, AI Blogs | Generated by Follow Builder
  </footer>

  <script>/* 全部 JS 内联 */</script>
</body>
</html>
```

### 完整 CSS 规范

```css
/* ===== 全局重置 ===== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; font-size: 16px; }
body {
  font-family: var(--font-family);
  background: var(--bg-primary);
  color: var(--text-primary);
  line-height: 1.7;
  min-height: 100vh;
}

/* ===== HERO ===== */
.hero {
  background: var(--hero-gradient);
  color: var(--text-hero);
  padding: var(--spacing-2xl) var(--spacing-lg);
  text-align: center;
  position: relative;
  overflow: hidden;
}
.hero::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle at 30% 50%, rgba(255,255,255,0.08) 0%, transparent 60%);
  animation: heroShine 8s ease-in-out infinite;
}
@keyframes heroShine {
  0%, 100% { transform: translate(0, 0); }
  50% { transform: translate(5%, 3%); }
}
.hero-content { position: relative; z-index: 1; max-width: 900px; margin: 0 auto; }
.hero-title {
  font-size: var(--font-size-hero);
  font-weight: 800;
  letter-spacing: -0.02em;
  margin-bottom: var(--spacing-sm);
  text-shadow: 0 2px 8px rgba(0,0,0,0.15);
}
.hero-date {
  font-size: var(--font-size-lg);
  opacity: 0.9;
  margin-bottom: var(--spacing-xs);
}
.hero-count {
  font-size: var(--font-size-xl);
  font-weight: 600;
  margin: var(--spacing-md) 0;
}
.hero-count strong {
  font-size: var(--font-size-2xl);
  font-weight: 800;
  color: var(--accent-gold);
}
.hero-stats {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: var(--spacing-sm) var(--spacing-lg);
  margin-top: var(--spacing-md);
  font-size: var(--font-size-sm);
}
.stat-item {
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(4px);
  border-radius: 20px;
  padding: 4px 14px;
  white-space: nowrap;
}

/* ===== 锚点导航 ===== */
.anchor-nav {
  position: sticky;
  top: 0;
  z-index: 100;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: var(--spacing-xs);
  padding: var(--spacing-sm) var(--spacing-md);
  background: rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 107, 53, 0.1);
  box-shadow: 0 1px 8px rgba(0,0,0,0.04);
}
.anchor-nav a {
  text-decoration: none;
  font-size: var(--font-size-sm);
  font-weight: 500;
  color: var(--text-secondary);
  padding: 6px 14px;
  border-radius: 18px;
  transition: var(--transition-fast);
  white-space: nowrap;
}
.anchor-nav a:hover,
.anchor-nav a:focus {
  background: var(--hero-gradient);
  color: #fff;
}

/* ===== 内容主区 ===== */
.content {
  max-width: 1100px;
  margin: 0 auto;
  padding: var(--spacing-2xl) var(--spacing-lg);
}

/* ===== 版块标题 ===== */
.section-block { margin-bottom: var(--spacing-2xl); }
.section-header {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  margin-bottom: var(--spacing-lg);
  padding-bottom: var(--spacing-sm);
  border-bottom: 3px solid;
  border-image: var(--hero-gradient) 1;
}
.section-header h2 {
  font-size: var(--font-size-xl);
  font-weight: 700;
  color: var(--text-primary);
}
.section-count {
  font-size: var(--font-size-sm);
  color: var(--text-muted);
  background: var(--bg-primary);
  padding: 2px 10px;
  border-radius: 12px;
}

/* ===== 卡片网格 ===== */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: var(--spacing-lg);
}

/* ===== 单条资讯卡片 ===== */
.card {
  background: var(--bg-card);
  border-radius: var(--card-radius);
  border: var(--card-border);
  box-shadow: var(--card-shadow);
  overflow: hidden;
  transition: var(--transition-normal);
  display: flex;
  flex-direction: column;
  opacity: 0;
  transform: translateY(24px);
}
.card.visible {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.card:hover {
  box-shadow: var(--card-shadow-hover);
  transform: translateY(-2px);
}
.card.visible:hover {
  transform: translateY(-2px);
}

.card-header {
  padding: var(--spacing-md) var(--spacing-lg) 0;
  display: flex;
  align-items: flex-start;
  gap: var(--spacing-sm);
}
.card-number {
  flex-shrink: 0;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--hero-gradient);
  color: #fff;
  font-size: var(--font-size-xs);
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-meta {
  display: flex;
  align-items: center;
  gap: var(--spacing-xs);
  flex-wrap: wrap;
  margin-bottom: 4px;
}
.card-title {
  font-size: var(--font-size-base);
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1.4;
}
.source-chip {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  font-size: var(--font-size-xs);
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 10px;
  white-space: nowrap;
}
.source-chip.x { background: #FFF0E6; color: #E8453A; }
.source-chip.podcast { background: #FFEBEE; color: #B71C1C; }
.source-chip.blog { background: #FFF3E0; color: #FF6B35; }

.card-body {
  padding: var(--spacing-sm) var(--spacing-lg);
  flex: 1;
  font-size: var(--font-size-sm);
  color: var(--text-secondary);
  line-height: 1.8;
  /* 不受字数限制，内容完整展示 */
}
.card-body p { margin-bottom: var(--spacing-xs); }
.card-body a {
  color: var(--accent-red);
  text-decoration: none;
  border-bottom: 1px dotted var(--accent-red);
}
.card-body a:hover { border-bottom-style: solid; }

.card-footer {
  padding: var(--spacing-sm) var(--spacing-lg) var(--spacing-md);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-sm);
  flex-wrap: wrap;
}
.card-time {
  font-size: var(--font-size-xs);
  color: var(--text-muted);
}
.card-link {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: var(--font-size-xs);
  font-weight: 600;
  color: #fff;
  background: var(--hero-gradient);
  padding: 6px 14px;
  border-radius: 16px;
  text-decoration: none;
  transition: var(--transition-fast);
}
.card-link:hover {
  opacity: 0.9;
  transform: scale(1.03);
}

/* ===== 页脚 ===== */
.footer {
  text-align: center;
  padding: var(--spacing-xl);
  font-size: var(--font-size-xs);
  color: var(--text-muted);
  border-top: 1px solid rgba(255, 107, 53, 0.12);
  margin-top: var(--spacing-2xl);
}

/* ===== 响应式 ===== */
@media (max-width: 768px) {
  :root {
    --font-size-hero: 1.8rem;
  }
  .card-grid {
    grid-template-columns: 1fr;
  }
  .hero-stats {
    gap: var(--spacing-xs);
  }
  .hero-stats .stat-item {
    font-size: var(--font-size-xs);
    padding: 3px 10px;
  }
  .anchor-nav {
    overflow-x: auto;
    justify-content: flex-start;
    flex-wrap: nowrap;
    -webkit-overflow-scrolling: touch;
  }
  .anchor-nav a {
    flex-shrink: 0;
  }
  .content {
    padding: var(--spacing-lg) var(--spacing-md);
  }
}
@media (max-width: 480px) {
  .card-grid {
    grid-template-columns: 1fr;
  }
  .hero { padding: var(--spacing-xl) var(--spacing-md); }
}
```

### JavaScript 规范

```js
// ===== 滚动渐入动画 =====
document.addEventListener('DOMContentLoaded', function() {
  const cards = document.querySelectorAll('.card');

  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry, index) => {
      if (entry.isIntersecting) {
        // 逐个渐入，产生级联效果
        setTimeout(() => {
          entry.target.classList.add('visible');
        }, index * 80);
        observer.unobserve(entry.target);
      }
    });
  }, {
    threshold: 0.1,
    rootMargin: '0px 0px -30px 0px'
  });

  cards.forEach(card => observer.observe(card));

  // ===== 锚点导航高亮 =====
  const sections = document.querySelectorAll('.section-block');
  const navLinks = document.querySelectorAll('.anchor-nav a');

  const navObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const id = entry.target.id;
        navLinks.forEach(link => {
          link.style.background = '';
          link.style.color = '';
          if (link.getAttribute('href') === '#' + id) {
            link.style.background = 'var(--hero-gradient)';
            link.style.color = '#fff';
          }
        });
      }
    });
  }, { threshold: 0.3 });

  sections.forEach(section => navObserver.observe(section));
});
```

### 北京人话时间格式化规则

将 ISO 时间戳转换为人类可读的北京时间格式。使用以下 JavaScript 函数：

```js
function formatBeijingTime(isoString) {
  if (!isoString) return '';

  const date = new Date(isoString);
  const now = new Date();

  // 确保使用北京时间 (UTC+8)
  const bjDate = new Date(date.toLocaleString('en-US', { timeZone: 'Asia/Shanghai' }));
  const bjNow = new Date(now.toLocaleString('en-US', { timeZone: 'Asia/Shanghai' }));

  const diffMs = bjNow - bjDate;
  const diffMins = Math.floor(diffMs / 60000);
  const diffHours = Math.floor(diffMs / 3600000);
  const diffDays = Math.floor(diffMs / 86400000);

  const hours = bjDate.getHours();
  const mins = bjDate.getMinutes().toString().padStart(2, '0');
  const timeStr = `${hours}:${mins}`;
  const period = hours < 12 ? '上午' : hours < 18 ? '下午' : '晚上';

  if (diffMins < 1) return '刚刚';
  if (diffMins < 60) return `${diffMins} 分钟前`;
  if (diffHours < 6) return `${diffHours} 小时前`;
  if (diffDays === 0) return `今天${period} ${timeStr}`;
  if (diffDays === 1) return `昨天${period} ${timeStr}`;
  if (diffDays < 7) return `${diffDays} 天前`;

  const month = bjDate.getMonth() + 1;
  const day = bjDate.getDate();
  return `${month}月${day}日 ${timeStr}`;
}
```

**关键规则：**
- 1 分钟内 → "刚刚"
- 1 小时内 → "X 分钟前"
- 6 小时内 → "X 小时前"
- 当天 → "今天上午/下午/晚上 HH:MM"
- 昨天 → "昨天上午/下午/晚上 HH:MM"
- 7 天内 → "X 天前"
- 更早 → "M月D日 HH:MM"

---

## 完整生成示例

以下是最终 HTML 晨报的卡片示例（单条资讯）：

```html
<article class="card">
  <div class="card-header">
    <span class="card-number">1</span>
    <div>
      <div class="card-meta">
        <span class="source-chip x">𝕏 X</span>
      </div>
      <h3 class="card-title">Sam Altman 暗示 GPT-5 将在今年秋季发布，将统一 o 系列和 GPT 系列</h3>
    </div>
  </div>
  <div class="card-body">
    <p>OpenAI CEO Sam Altman 在最新的 X 帖子中回应社区提问时暗示，GPT-5 将在 2026 年秋季发布。他表示，新模型将统一目前的 o 系列推理模型和 GPT 系列，不再需要用户手动选择模型。他特别强调："我们正在让模型自己决定何时需要深度思考，而不是让用户来做这个选择。"</p>
    <p>这一消息在 AI 社区引发热议。Andrej Karpathy 转发评论称这是"正确的架构决策"。社区普遍认为这标志着 OpenAI 正在将推理能力从"用户可选模式"升级为"模型内置能力"。</p>
  </div>
  <div class="card-footer">
    <span class="card-time">2 小时前</span>
    <a href="https://x.com/sama/status/1234567890" class="card-link" target="_blank" rel="noopener noreferrer">查看原文 →</a>
  </div>
</article>
```

---

## 配置管理

沿用原版 follow-builders 的配置系统。配置文件位置：`~/.follow-builders/config.json`。

可通过对话修改：
- 频率、时间、时区
- 语言（简体中文 / 中英双语）
- 发送方式

**源列表由中心管理，不可修改。** 如需建议添加源，前往 https://github.com/zarazhangrui/follow-builders 提 issue。

---

## 手动触发

当用户键入 `/ai` 或要求 "来一份晨报" 时：
1. 跳过 cron 检查，立即运行生成流程
2. 告知用户 "正在生成本日晨报，大约需要一分钟..."
3. 生成 HTML 文件并调用 `present_files` 展示

---

## 技能维护

改进自 Zara Zhang 的 [Follow Builders](https://github.com/zarazhangrui/follow-builders)，核心数据管线归功于原作者。本改进版新增：
- HTML 晨报视觉输出
- 五大版块分类体系
- 北京人话时间格式
- 响应式卡片网格与滚动动画
- 纯内联 HTML/CSS/JS 架构

如有问题或建议，欢迎反馈。
