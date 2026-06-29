# Follow Builder — AI Builders Morning Report

> **基于 [Zara Zhang 的 Follow Builders](https://github.com/zarazhangrui/follow-builders) 技能深度改进而来。**
> 原项目核心理念："Follow Builders, Not Influencers" — 追踪真正在做事的人。

[![Version](https://img.shields.io/badge/version-2.0.0-orange)](https://github.com/AARONROKIE/followbuilder)
[![Based on](https://img.shields.io/badge/based%20on-follow--builders%20v1.x-blue)](https://github.com/zarazhangrui/follow-builders)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## 改进说明

本项目在 Zara Zhang 原版 **follow-builders** 技能的数据管线基础上，进行了以下重大升级：

| # | 改进项 | 说明 |
|---|--------|------|
| 1 | **HTML 晨报输出** | 纯 HTML/CSS/JS 单文件，橙红渐变 Hero，响应式卡片网格 |
| 2 | **全局连续编号** | 编号贯穿全文，不按版块重置，一目了然今日总条数 |
| 3 | **五大内容版块** | 模型与前沿、产品与发布、商业与投资、工程与工具、深度播客 |
| 4 | **锚点导航** | 顶部 sticky 导航栏，一键跳转到各版块 |
| 5 | **北京人话时间** | "刚刚" / "2 小时前" / "今天上午 09:48" 替代 ISO 时间戳 |
| 6 | **滚动渐入动画** | Intersection Observer 驱动，卡片逐个淡入 |
| 7 | **不受字数限制** | 每条资讯卡片内容完整呈现，不截断 |
| 8 | **来源 Chip + 跳转按钮** | X / Podcast / Blog 彩色标签 + target="_blank" 外链 |
| 9 | **系统字体栈** | 零外部依赖，CSS 变量驱动的完整设计系统 |
| 10 | **工作区本地保存** | HTML 晨报保存为本地文件，浏览器可直接打开 |

### 保留的原版功能
- 中心 feed 数据管线（无需用户提供 API key）
- 多平台支持（OpenClaw / 其他 Agent）
- 播客 / X / 博客三源聚合
- Telegram / Email / stdout 多种发送方式
- 可定制的提示词系统

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| **v2.0.0** | 2026-06-29 | 首次发布改进版，新增 HTML 晨报输出、五大版块分类、北京人话时间格式等 |
| v1.x | 2026 | Zara Zhang 原始版本，纯文本 digest 输出 |

## 安装

### 前置要求
- Node.js 22+
- 可选：Telegram Bot Token（Telegram 发送）
- 可选：Resend API Key（Email 发送）

### 安装依赖
```bash
cd scripts && npm install
```

### WorkBuddy 用户
将本技能放入 `~/.workbuddy/skills/followbuilder/` 目录，在对话中说 "来一份 AI 晨报" 即可触发。

## 使用方法

### 手动触发
```
来一份 AI 晨报
```

### 定时发送（OpenClaw）
```bash
openclaw cron add \
  --name "AI Builders Morning Report" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "Run followbuilder skill" \
  --announce
```

## 项目结构

```
followbuilder/
├── SKILL.md                  # WorkBuddy 技能定义（核心）
├── templates/
│   └── morning-report.html   # HTML 晨报参考模板
├── config/
│   ├── config-schema.json    # 用户配置 JSON Schema
│   └── default-sources.json  # 默认追踪源
├── prompts/
│   ├── digest-intro.md       # 晨报整体格式指引
│   ├── summarize-tweets.md   # 推文摘要提示词
│   ├── summarize-podcast.md  # 播客摘要提示词
│   ├── summarize-blogs.md    # 博客摘要提示词
│   └── translate.md          # 翻译提示词
└── scripts/
    ├── prepare-digest.js     # 数据准备脚本
    ├── deliver.js            # 发送脚本
    ├── package.json
    └── package-lock.json
```

## 致谢

- [Zara Zhang](https://github.com/zarazhangrui) — [follow-builders](https://github.com/zarazhangrui/follow-builders) 原作者，提供了核心数据管线和 "Follow Builders" 哲学
- [WorkBuddy](https://www.codebuddy.cn) — AI 工作助手平台

## 许可

MIT License
