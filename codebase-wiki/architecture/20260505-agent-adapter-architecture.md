---
id: A-001
title: Agent Adapter 体系架构详解
description: >
  梳理 Open Design 项目的 agent adapter 层设计：如何检测、适配和驱动 15 种
  code agent CLI，以及四大流协议（claude-stream-json / json-event-stream /
  acp-json-rpc / plain）的解析架构。
category: architecture
created: 2026-05-05
updated: 2026-05-06
tags: [agent, adapter, harness, architecture, acp, streaming]
status: draft
references:
  - id: R-001
    rel: derives
    file: ../reference/20260505-agent-adapter-quick-reference.md
  - id: A-002
    rel: related-to
    file: ./20260505-preview-rendering-architecture.md
  - id: A-003
    rel: related-to
    file: ./20260506-opencode-local-cli-architecture.md
---

# Agent Adapter 体系架构详解

> Open Design 不自己实现 agent loop，而是将整个 agent 循环（模型调用、工具使用、
> 上下文管理、权限处理）委托给用户本机已安装的 code agent CLI。OD 的角色是
> **检测 → 喂入 → 流式解析**。

## 核心设计原则

Adapter 层是 OD 最核心的设计决策（`docs/agent-adapters.md`）：

- **不重新实现 agent loop**：code agent 领域已收敛到几个强实现（Claude Code、Codex、Devin 等），重新实现不如对接所有
- **PATH 扫描自动检测**：灵感来自 [multica](https://github.com/multica-ai/multica)
- **能力驱动的 UI 降级**：检测每个 agent 的能力，缺少某能力时 UI 自动隐藏对应功能

## 三层架构

```
┌──────────────── Web App (Next.js 16) ────────────────┐
│  chat pane │ artifact tree │ preview │ comment/slider │
│                    session bus                        │
│              Transport (SSE / api-direct)             │
└──────────────────────┬───────────────────────────────┘
                       │
┌──────────────── Daemon ──────────────────────────────┐
│  session manager   │ agent adapter pool              │
│  skill registry    │ design-system resolver           │
│  artifact store    │ preview compile pipeline         │
└──────┬───────────────────────────────────────────────┘
       │
┌──── Agent CLIs ────┐
│ claude, codex,     │
│ devin, gemini,     │
│ opencode, cursor,  │
│ copilot, pi, ...   │
└────────────────────┘
```

## Agent 检测机制

实现位于 `apps/daemon/src/agents.ts`。

### 检测流程（`detectAgents()`）

1. 遍历 `AGENT_DEFS` 数组（15 个 agent 定义）
2. 对每个 agent 并行执行 `probe()`：
   - **PATH 解析**（`resolveAgentExecutable()`）：扫描 `$PATH` + 用户工具链目录（`~/.local/bin`、`~/.cargo/bin`、nvm/fnm/mise 版本目录等）
   - **版本探测**：执行 `<bin> --version`
   - **能力嗅探**：对支持的 agent（如 Claude Code），执行 `<bin> -p --help` 检测 `--include-partial-messages`、`--add-dir` 等可选 flag
   - **模型列表获取**：通过 `listModels`（CLI 子命令）或 `fetchModels`（ACP 握手）获取可用模型
3. 返回各 agent 的 `available`、`version`、`path`、`models` 等信息

### 关键函数

| 函数 | 位置 | 职责 |
|------|------|------|
| `detectAgents()` | `agents.ts:921` | 并行探测所有 agent |
| `probe()` | `agents.ts:853` | 单个 agent 的完整探测 |
| `resolveAgentExecutable()` | `agents.ts:810` | 解析 bin + fallbackBins |
| `resolveOnPath()` | `agents.ts:790` | PATH + 工具链目录扫描 |
| `fetchModels()` | `agents.ts:823` | 获取 agent 可用模型列表 |

## 四大流协议

Daemon 根据 `streamFormat` 字段选择解析器，将各 CLI 的 stdout 映射为统一的 UI 事件：

### 1. `claude-stream-json`

- **适用**：Claude Code
- **格式**：JSON Lines（`--output-format stream-json`）
- **事件**：text / thinking / tool_use / tool_result / status
- **特点**：最丰富的事件类型，是参考实现

### 2. `json-event-stream`

- **适用**：Codex、Gemini CLI、OpenCode、Cursor Agent（各有专属 parser）
- **格式**：JSON 事件流
- **特点**：通过 `eventParser` 字段区分不同 CLI 的解析逻辑

### 3. `acp-json-rpc`

- **适用**：Devin、Hermes、Kimi、Kiro、Kilo、Vibe（6 个 agent）
- **格式**：Agent Client Protocol — JSON-RPC over stdio
- **生命周期**：`initialize → session/new → session/prompt`
- **模型发现**：通过 `detectAcpModels()` (`apps/daemon/src/acp.ts`) 在握手阶段获取

### 4. `plain`

- **适用**：Qwen Code、DeepSeek TUI
- **格式**：原始文本，逐 chunk 转发
- **限制**：无结构化 tool call 事件

### 5. 其他

- `copilot-stream-json`：GitHub Copilot CLI 专用，JSONL 格式类似 claude-stream
- `pi-rpc`：Pi 专用 RPC 协议，支持 reasoning 档位

## Prompt 传递策略

| 策略 | 标记 | 适用 Agent | 原因 |
|------|------|-----------|------|
| stdin pipe | `promptViaStdin: true` | Claude, Codex, Gemini, OpenCode, Cursor, Pi, Devin 等 | 避免 Windows `ENAMETOOLONG`（CreateProcess 32KB 限制）和 Linux `E2BIG` |
| argv 参数 | `promptViaStdin: false` | Copilot, DeepSeek | CLI 不支持 stdin sentinel |

对于 argv 传递的 agent，通过 `maxPromptArgBytes` + 三重预算检查（raw 字节 / cmd shim 展开 / direct exe 展开）防止超限。

## Skill 注入策略

三种方式，优先级递减（`docs/agent-adapters.md` §4）：

1. **原生 skill 加载**：symlink 到 `~/.<agent>/skills/`，agent 自动加载。Claude Code、部分 Codex 版本支持
2. **Prompt 注入**：将 SKILL.md + references 拼入 system prompt。所有 agent 通用，但消耗更多 token
3. **文件放置**：写入 `.cursorrules` 或 `AGENTS.md` 到项目目录。Cursor Agent 等支持

## 三种部署拓扑

| 拓扑 | 描述 | Agent 运行位置 |
|------|------|---------------|
| A — 完全本地 | daemon + web 都在用户机器 | 本地 |
| B — Web on Vercel + 本地 daemon | 通过 tunnel 连接 | 本地 |
| C — 纯 Vercel + 直接 API | 无 CLI，降级体验 | Anthropic API（无 agent CLI） |

## 权限模型

OD **继承**底层 agent 的权限模型，不自建沙箱：

- Claude Code：`--permission-mode bypassPermissions`（无 TTY 环境必须）
- Codex：`--sandbox workspace-write`
- Devin：`--permission-mode dangerous --respect-workspace-trust false`
- API fallback：白名单 Read/Write/Edit，限制在 artifact cwd

## 关键源文件

| 文件 | 职责 |
|------|------|
| `apps/daemon/src/agents.ts` | 所有 agent 定义、检测、PATH 解析、模型验证 |
| `apps/daemon/src/acp.ts` | ACP JSON-RPC 协议实现与模型发现 |
| `apps/daemon/src/pi-rpc.ts` | Pi RPC 模型解析 |
| `docs/agent-adapters.md` | adapter 架构设计文档（305 行） |
| `docs/architecture.md` | 系统整体架构文档（339 行） |
