---
id: R-001
title: Agent 适配器速查表
description: >
  Open Design 支持的 15 个 agent CLI 适配器一览：二进制名、流格式、
  prompt 传递方式、默认模型与关键 flags。
category: reference
created: 2026-05-05
updated: 2026-05-05
tags: [agent, adapter, reference, quick-reference]
status: draft
references:
  - id: A-001
    rel: derived-from
    file: ../architecture/20260505-agent-adapter-architecture.md
---

# Agent 适配器速查表

> 所有 agent 定义位于 `apps/daemon/src/agents.ts:119-713`（`AGENT_DEFS` 数组）。

## 完整 Agent 列表

| # | ID | 名称 | 二进制 | Fallback | 流格式 | Prompt | 备注 |
|---|---|---|---|---|---|---|---|
| 1 | `claude` | Claude Code | `claude` | `openclaude` | `claude-stream-json` | stdin | 参考实现；探测 `--include-partial-messages`、`--add-dir` |
| 2 | `codex` | Codex CLI | `codex` | — | `json-event-stream` (codex) | stdin | 支持 reasoning 档位（minimal/low/medium/high） |
| 3 | `devin` | Devin for Terminal | `devin` | — | `acp-json-rpc` | ACP RPC | `--permission-mode dangerous` |
| 4 | `gemini` | Gemini CLI | `gemini` | — | `json-event-stream` (gemini) | stdin | env: `GEMINI_CLI_TRUST_WORKSPACE=true` |
| 5 | `opencode` | OpenCode | `opencode` | — | `json-event-stream` (opencode) | stdin | `opencode models` 列出模型 |
| 6 | `hermes` | Hermes | `hermes` | — | `acp-json-rpc` | ACP RPC | 支持 MCP discovery (`mature-acp`) |
| 7 | `kimi` | Kimi CLI | `kimi` | — | `acp-json-rpc` | ACP RPC | 支持 MCP discovery (`mature-acp`) |
| 8 | `cursor-agent` | Cursor Agent | `cursor-agent` | — | `json-event-stream` (cursor-agent) | stdin | workspace 模式，`--force --trust` |
| 9 | `qwen` | Qwen Code | `qwen` | — | `plain` | stdin | Gemini CLI fork，`--yolo` |
| 10 | `copilot` | GitHub Copilot CLI | `copilot` | — | `copilot-stream-json` | argv | `--allow-all-tools`；支持 `--add-dir` |
| 11 | `pi` | Pi | `pi` | — | `pi-rpc` | RPC | `--mode rpc --no-session`；支持 reasoning |
| 12 | `kiro` | Kiro CLI | `kiro-cli` | — | `acp-json-rpc` | ACP RPC | — |
| 13 | `kilo` | Kilo | `kilo` | — | `acp-json-rpc` | ACP RPC | — |
| 14 | `vibe` | Mistral Vibe CLI | `vibe-acp` | — | `acp-json-rpc` | ACP RPC | — |
| 15 | `deepseek` | DeepSeek TUI | `deepseek` | — | `plain` | argv | `maxPromptArgBytes: 30000` |

## 流格式分组

| 流格式 | Agent | 解析入口 |
|---|---|---|
| `claude-stream-json` | Claude Code | `claude-stream.ts` |
| `json-event-stream` | Codex, Gemini, OpenCode, Cursor Agent | 各自 `eventParser` |
| `acp-json-rpc` | Devin, Hermes, Kimi, Kiro, Kilo, Vibe | `acp.ts` |
| `copilot-stream-json` | Copilot | `copilot-stream.ts` |
| `pi-rpc` | Pi | `pi-rpc.ts` |
| `plain` | Qwen, DeepSeek | 逐 chunk 转发 |

## 默认模型速查

| Agent | 默认模型提示 |
|---|---|
| Claude Code | sonnet / opus / haiku / claude-opus-4-5 / claude-sonnet-4-5 / claude-haiku-4-5 |
| Codex | gpt-5-codex / gpt-5 / o3 / o4-mini |
| Devin | adaptive / swe / opus / sonnet / codex / gpt / gemini |
| Gemini CLI | gemini-2.5-pro / gemini-2.5-flash |
| OpenCode | anthropic/claude-sonnet-4-5 / openai/gpt-5 / google/gemini-2.5-pro |
| Hermes | openai-codex:gpt-5.5 / gpt-5.4 / gpt-5.4-mini |
| Kimi | kimi-k2-turbo-preview / moonshot-v1-8k / moonshot-v1-32k |
| Cursor Agent | auto / sonnet-4 / sonnet-4-thinking / gpt-5 |
| Qwen | qwen3-coder-plus / qwen3-coder-flash |
| Copilot | claude-sonnet-4.6 / gpt-5.2 |
| Pi | anthropic/claude-sonnet-4-5 / openai/gpt-5 / google/gemini-2.5-pro 等 |
| DeepSeek | deepseek-v4-pro / deepseek-v4-flash |

## Reasoning 支持

仅以下 agent 暴露 reasoning 档位选择：

| Agent | 档位 |
|---|---|
| Codex | default / minimal / low / medium / high |
| Pi | default / off / minimal / low / medium / high / xhigh |

Codex 还有模型级 reasoning 钳位逻辑（`clampCodexReasoning()`，`agents.ts:82-99`）：
- gpt-5.2/5.3/5.4/5.5：`minimal` → `low`
- gpt-5.1：`xhigh` → `high`
- gpt-5.1-codex-mini：只接受 `medium` / `high`

## 关键 CLI flags 速查

| Agent | 非交互/自动审批 flag | 其他关键 flag |
|---|---|---|
| Claude Code | `--permission-mode bypassPermissions` | `--output-format stream-json --verbose` |
| Codex | `--sandbox workspace-write` | `--skip-git-repo-check --json` |
| Devin | `--permission-mode dangerous` | `--respect-workspace-trust false` |
| Gemini | `--yolo` | `--output-format stream-json` |
| OpenCode | `--dangerously-skip-permissions` | `--format json` |
| Cursor Agent | `--force --trust` | `--print --output-format stream-json --stream-partial-output` |
| Copilot | `--allow-all-tools` | `--output-format json` |
| DeepSeek | `--auto` | `exec` 子命令 |
