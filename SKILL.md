---
name: dev-log
description: Use this skill when an AI coding agent should read or write Markdown development logs or temporary handoff snapshots in the current repository. Trigger when the user asks to 记录日志, write a dev log, continue previous work, switch agents, generate handoff/current state, document decisions, blockers, verification, or preserve handoff context for Codex, Claude Code, OpenCode, and other coding agents.
---

# Dev Log Skill

## 目标

在使用 AI agent 开发代码时，用 Markdown 开发日志保存关键上下文。日志既给人类回顾，也给后续 agent 接力。默认日志可以提交到 git，因此必须克制、准确、脱敏。

这个 skill 是通用协议，不绑定 Codex、Claude Code、OpenCode 或其他特定 agent。只要 agent 能读取 Markdown 指令和项目文件，就可以使用。

## 默认约定

如果项目没有配置文件，使用以下默认值：

- 日志目录：`docs/dev-log`
- 文件模式：按天记录，文件名为 `YYYY-MM-DD.md`
- 语言：中文优先，除非项目明显使用其他语言
- 读者：`hybrid`，兼顾人类阅读和 agent 接力
- 触发策略：`conservative`，只在关键节点自动记录
- 写入策略：append-only，追加新条目，不静默改旧条目
- 隐私策略：日志默认可提交到 git，禁止原样记录敏感信息
- 交接快照：默认写入 `.agents/handoff/current.md`，临时、可覆盖、默认不提交

## 开始开发前

当任务是中等以上复杂度的开发、修 bug、重构、继续之前任务、解释历史决策时：

1. 先查找配置文件：优先 `.devlog.yml`，其次 `docs/dev-log/config.yml`。
2. 确认日志目录和读取策略。
3. 读取最近相关日志，默认读取最近 1 到 3 天，最多 8 条。
4. 按当前任务关键词、文件路径、模块名、错误信息搜索相关日志。
5. 只把高价值结论带入上下文。
6. 永远以当前代码和测试结果为准，日志只是参考线索。

简单问答、一行小改、纯格式化任务通常不需要读取日志。

当用户要求“继续上次任务”且项目存在 `.agents/handoff/current.md` 时，先读取交接快照，再按快照引用读取相关 dev log 和当前代码。交接快照是当前状态提示，不是事实来源；涉及实现细节时仍以代码、diff 和验证结果为准。

## 什么时候写日志

默认 `conservative` 策略下，在这些场景记录：

- 用户明确要求记录日志、写 dev log、保存开发过程。
- 做出重要技术决策或改变实现方案。
- 完成跨多个文件或模块的一组改动。
- 发现阻塞原因、测试失败根因或重要约束。
- 任务完成，并有明确验证结果。
- 准备暂停、切换任务，或上下文可能中断。

这些场景通常不记录：

- 简单问答。
- 单行小改、纯格式化、无行为变化的清理。
- 只读代码但没有形成可复用结论。
- 常规命令成功执行，但没有产生额外决策或风险。

自动记录前先判断价值：如果这条日志不能帮助未来的人或 agent 节省大约 5 分钟以上，就不要自动写入。

## 用户主动要求记录时

当用户明确要求记录日志时，默认尝试记录，但仍需判断重复性、价值和敏感信息：

1. 读取当天日志最近几条，默认 5 条。
2. 判断是否已有等价记录。
3. 如果重复，不追加，简短告诉用户已有等价记录。
4. 如果有新增事实，追加一条简短 update。
5. 写入前完成脱敏检查。
6. 写完后告诉用户日志路径。

## 如何写日志

默认追加到：

```text
docs/dev-log/YYYY-MM-DD.md
```

如果当天文件不存在，创建文件并添加标题：

```md
# Dev Log - YYYY-MM-DD
```

默认条目结构：

```md
## YYYY-MM-DD HH:mm TZ - 标题

Agent: Codex | Claude Code | OpenCode | Other
Status: in_progress | done | blocked

### Summary
用 1 到 3 句话说明任务背景、关键进展或结论。

### Decisions
- 记录重要技术决策和原因。

### Changes
- `path/to/file`: 概括关键改动。

### Verification
- `command`: passed | failed | not run，并说明重要结果。

### Next Steps
- 留给人类或下一个 agent 的接力信息。
```

根据实际情况省略空章节。默认单条日志控制在 20 到 40 行。

## Agent 交接快照

当用户明确表示要切换 agent、暂停当前任务、生成 handoff、整理当前开发状态，或让另一个 agent 接着做时，生成临时交接快照，而不是追加开发日志。

默认写入：

```text
.agents/handoff/current.md
```

交接快照用于回答“当前目标是什么、已经做到哪、下一个 agent 应该从哪里继续”。它不是长期日志，默认可覆盖，默认不提交到 git。写入前应读取当前任务上下文、当前代码状态、git diff 摘要、最近相关 dev log，并只保留对接手有帮助的信息。

默认结构：

```md
# Agent Handoff - 当前开发状态

Updated: YYYY-MM-DD HH:mm TZ
From Agent: Codex | Claude Code | OpenCode | Other
Suggested Next Agent: Claude Code | Codex | OpenCode | Other | Unknown
Status: in_progress | blocked | ready_for_next

## Current Goal
当前开发目标。

## Progress
已经完成的工作和当前进度。

## Current Context
用户要求、边界条件、重要约束。

## Decisions So Far
- 已确认的技术决策和原因。

## Changed Files
- `path/to/file`: 当前改动状态和注意点。

## Verification
- `command`: passed | failed | not run，并说明重要结果。

## Open Questions
- 尚未确认的问题。

## Next Steps
1. 下一个 agent 应该先做什么。
2. 然后做什么。

## Relevant Dev Logs
- `docs/dev-log/YYYY-MM-DD.md`: 相关历史线索。
```

根据实际情况省略空章节。不要复制聊天全文，不要粘贴大段 diff，不要记录未经脱敏的敏感信息。写完后告诉用户交接文件路径。

## 去重、更新和纠错

- 优先追加新条目，不直接改旧条目。
- 写入前比较当天最近几条，避免重复记录同一事实。
- 旧记录需要补充时，追加 update 条目。
- 旧记录有误时，追加 correction 条目，说明旧判断、当前事实和验证来源。
- 只有用户明确要求整理、重写、清理日志时，才可以编辑旧日志。

## 隐私和脱敏

日志默认可以提交到 git。禁止原样记录：

- API key、token、cookie、session、JWT。
- 密码、私钥、证书内容。
- 真实手机号、身份证号、银行卡号、详细住址。
- 生产用户原始数据。
- 带签名参数或敏感查询参数的 URL。
- `.env` 中的敏感值、数据库连接串、内部服务密钥。

可以记录脱敏后的事实：

- 使用了 `DASHSCOPE_API_KEY` 环境变量，但未记录具体值。
- 生产 API 返回 401，确认是 token 过期导致。
- 数据库连接串已脱敏为 `postgres://user:***@host/db`。

如果无法安全脱敏，不要写入可提交日志，先向用户说明风险。

交接快照虽然默认不提交，但仍按同样标准脱敏。除非用户明确要求并确认风险，不要把临时凭证、真实用户数据、生产原始响应或完整私密配置写入 handoff。

## 配置

优先读取 `.devlog.yml`，其次读取 `docs/dev-log/config.yml`。配置缺失时使用默认值。未知字段忽略，非法字段回退默认值，不要因为配置问题阻塞开发。

完整配置见 `references/configuration.md`。触发策略见 `references/trigger-policy.md`。读写和交接规则见 `references/read-write-policy.md`。脱敏规则见 `references/privacy-redaction.md`。

## 最终回复

如果本轮写入了日志或交接快照，最终回复里简短说明文件路径。如果判断无需记录，也不需要特意解释，除非用户明确要求记录但被去重或脱敏规则拦截。
