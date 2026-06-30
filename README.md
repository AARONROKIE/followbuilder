# Follow Builder — AI Builders Morning Report 📰

> **Follow Builders, Not Influencers** — 追踪 AI 领域真正的建设者，每日生成橙红渐变晨报风 HTML 日报。

基于 [Zara Zhang 的 Follow Builders](https://github.com/zarazhangrui/follow-builders) 进行深度改进。

---

## ✨ 改进亮点

本改进版在保留原版核心数据管线（X/Twitter 推文、YouTube 播客、AI 博客自动抓取）的基础上，新增以下能力：

| # | 改进项 | 说明 |
|---|--------|------|
| 1 | **HTML 晨报输出** | 纯 HTML/CSS/JS 单文件，橙红渐变 Hero，响应式卡片网格 |
| 2 | **编号贯穿全文** | 全局连续序号，一目了然今日总条数 |
| 3 | **六大内容版块** | 模型与前沿、产品与发布、商业与投资、工程与工具、深度播客、股票市场分析 |
| 4 | **锚点导航** | 顶部固定导航栏，一键跳转到各版块 |
| 5 | **北京人话时间** | "今天上午 09:48" / "2 小时前" / "昨天下午 15:30" |
| 6 | **滚动渐入动画** | Intersection Observer 驱动，卡片逐个淡入 |
| 7 | **系统字体栈** | CSS 变量驱动，零外部依赖 |
| 8 | **本地 HTML 文件** | 保存至工作区，可在浏览器直接打开 |
| 9 | **📈 股票市场分析** | **新增！** 整合同花顺问财、通达信、NeoData、WeStock 等金融数据源，覆盖 AI 板块行情、资金流向、券商研报、市值动态 |

---

## 📈 股票市场分析功能（v2.0 新增）

### 覆盖范围

| 市场 | 标的 | 数据维度 |
|------|------|----------|
| 🇺🇸 美股 | NVDA、MSFT、GOOGL、AMZN、META、AMD、SMCI 等 | 行情、资金流向、财报 |
| 🇨🇳 A股 | AI 算力、大模型概念、AI 应用板块 | 涨跌幅、成交量、龙虎榜 |
| 🇭🇰 港股 | 腾讯、阿里、百度、商汤等 | 行情、南向资金 |
| 🌐 产业链 | 芯片/半导体、云计算、AI 应用 | 产业链动态、券商研报 |

### 数据源

- **同花顺问财（iWenCai）** — A股/港股/美股行情、资金流向、板块行情
- **通达信（TDX）** — K线数据、技术指标、个股筛选
- **NeoData** — 自然语言金融数据查询
- **腾讯自选股（WeStock）** — 行情、研报、新闻、公告

### 示例卡片

```html
<article class="card">
  <div class="card-header">
    <span class="card-number">15</span>
    <div>
      <div class="card-meta">
        <span class="source-chip stock-chip">📈 股票</span>
      </div>
      <h3 class="card-title">英伟达（NVDA）涨 4.2%，突破 150 美元创历史新高</h3>
    </div>
  </div>
  <div class="card-body">
    <p>英伟达股价今日大涨 4.2% 至 152.37 美元，市值突破 3.75 万亿美元。
    驱动因素包括：B200 GPU 大规模出货预期、多家投行上调目标价至 180-200 美元区间。
    摩根士丹利研报指出，AI 算力需求仍在加速，2026 年数据中心 GPU 市场规模有望突破 1500 亿美元。</p>
  </div>
  <div class="card-footer">
    <span class="card-time">今天下午 15:30</span>
    <a href="#" class="card-link" target="_blank">查看详情 →</a>
  </div>
</article>
```

---

## 🏗️ 六大内容版块

| 版块 | 覆盖内容 |
|------|----------|
| 🤖 模型与前沿 | 模型发布、论文解读、基准测试、前沿突破 |
| 🚀 产品与发布 | 产品发布、功能更新、新工具、SDK |
| 💼 商业与投资 | 融资、收购、战略调整、行业趋势 |
| 🔧 工程与工具 | 开发实践、基础设施、开源项目、架构 |
| 🎙️ 深度播客 | 播客长内容、深度访谈、技术长文 |
| 📈 股票市场分析 | AI 板块行情、上市公司股价、资金流向、券商研报、市值动态 |

---

## 🚀 快速开始

### 安装为 WorkBuddy Skill

```bash
git clone https://github.com/AARONROKIE/followbuilder.git ~/.workbuddy/skills/followbuilder
```

### 首次运行

在 WorkBuddy 中：
1. 键入 `/ai` 或说 "来一份晨报"
2. 技能会自动检测平台并引导完成设置
3. 设置完成后可配置每日/每周自动推送

### 配置文件

用户配置存储在 `~/.follow-builders/config.json`：

```json
{
  "language": "zh",
  "frequency": "daily",
  "timezone": "Asia/Shanghai",
  "deliveryTime": "08:00",
  "delivery": {
    "method": "stdout"
  }
}
```

---

## 📁 项目结构

```
followbuilder/
├── SKILL.md                  # 技能主文件（WorkBuddy 入口）
├── README.md                 # 本文件
├── prompts/                  # LLM 提示词模板
│   ├── digest-intro.md       # 日报编排规则
│   ├── summarize-tweets.md   # 推文摘要规则
│   ├── summarize-podcast.md  # 播客摘要规则
│   ├── summarize-blogs.md    # 博客摘要规则
│   ├── summarize-stocks.md   # 股票分析规则 🆕
│   └── translate.md          # 翻译规则
├── templates/                # HTML 模板
│   └── morning-report.html   # 晨报 HTML 模板
├── scripts/                  # 数据准备与投递脚本
│   ├── prepare-digest.js     # 聚合 feed 数据
│   └── deliver.js            # Telegram/Email 投递
├── config/                   # 默认配置
│   ├── default-sources.json  # 追踪的 builders 列表
│   └── config-schema.json    # 配置格式
└── feed-*.json               # 本地 feed 缓存
```

---

## 📝 许可与致谢

本项目基于 [Zara Zhang 的 Follow Builders](https://github.com/zarazhangrui/follow-builders) 改进。
核心数据管线归功于原作者 Zara Zhang。

改进部分（HTML 晨报视觉输出、六大版块分类体系、股票市场分析板块等）由 Aaron (AARONROKIE) 贡献。

---

## 🔗 链接

- 原作者仓库: [github.com/zarazhangrui/follow-builders](https://github.com/zarazhangrui/follow-builders)
- 本改进版: [github.com/AARONROKIE/followbuilder](https://github.com/AARONROKIE/followbuilder)
