---
layout: home

hero:
  name: "Open Design Wiki"
  text: "源码阅读与概念笔记"
  tagline: 架构分析 · 技术讨论 · Issue 记录 · 参考手册 · 路线图
  actions:
    - theme: brand
      text: 文档索引
      link: /INDEX#文档索引
    - theme: alt
      text: 书写规范
      link: /CONVENTIONS

features:
  - title: 架构分析
    details: 模块职责、依赖与系统设计笔记。
    link: /INDEX
  - title: 技术讨论
    details: 方案对比、概念辨析与深度笔记。
    link: /INDEX
  - title: Issue 记录
    details: AI coding 过程中的问题现象、修复过程与回归结论。
    link: /INDEX
  - title: 参考手册
    details: 目录结构与速查。
    link: /INDEX
  - title: 规划路线
    details: 差距分析、优先级与待办。
    link: /INDEX
---

> 本目录（`codebase-wiki/`）存放 AI 辅助生成的分析文档、技术讨论、Issue 记录、参考手册与规划路线。  
> 书写规范请参考 [CONVENTIONS.md](./CONVENTIONS.md)（也可在仓库中直接打开该文件）。

在分类子目录下添加首篇文档后，在仓库根目录运行 skill 自带的 `regenerate-sidebar.mjs` 以更新侧栏与导航。

## 文档索引

### architecture/ — 架构分析

| # | 文件 | 标题 | 概述 |
|---|------|------|------|
| A-001 | [20260505-agent-adapter-architecture.md](./architecture/20260505-agent-adapter-architecture.md) | Agent Adapter 体系架构详解 | agent adapter 层设计：检测、适配和驱动 15 种 code agent CLI，四大流协议解析架构 |
| A-002 | [20260505-preview-rendering-architecture.md](./architecture/20260505-preview-rendering-architecture.md) | Preview 渲染体系架构详解 | AI 生成代码的预览全链路：artifact 解析、iframe 沙箱渲染、五种渲染器、安全隔离与 Live Artifact |
| A-003 | [20260506-opencode-local-cli-architecture.md](./architecture/20260506-opencode-local-cli-architecture.md) | Local CLI Agent 连接架构详解 | 本地 CLI agent 检测与连接：PATH 扫描、/api/chat 请求、spawn 子进程、stdin prompt 注入与 JSON 流解析 |

### discussion/ — 技术讨论

| # | 文件 | 标题 | 概述 |
|---|------|------|------|
|  |  |  |  |

### issue/ — Issue 记录

| # | 文件 | 标题 | 概述 |
|---|------|------|------|
|  |  |  |  |

### reference/ — 参考手册

| # | 文件 | 标题 | 概述 |
|---|------|------|------|
| R-001 | [20260505-agent-adapter-quick-reference.md](./reference/20260505-agent-adapter-quick-reference.md) | Agent 适配器速查表 | 15 个 agent CLI 适配器一览：二进制名、流格式、prompt 传递方式、默认模型与关键 flags |

### roadmap/ — 规划路线

| # | 文件 | 标题 | 概述 |
|---|------|------|------|
|  |  |  |  |
