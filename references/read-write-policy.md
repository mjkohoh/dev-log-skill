# 读写策略

## 读取流程

复杂开发任务开始前，按以下顺序读取日志：

1. 查找 `.devlog.yml`。
2. 如果不存在，查找 `docs/dev-log/config.yml`。
3. 如果没有配置，使用默认值。
4. 读取最近 1 到 3 天日志，默认最多 8 条。
5. 用任务关键词、文件路径、模块名、错误信息搜索相关日志。
6. 只摘取高价值结论进入上下文。

日志是参考线索，不是当前事实来源。涉及实现细节时，必须打开当前代码或运行验证。

如果用户要求继续上次任务、切换 agent 后接手，或项目存在 `.agents/handoff/current.md` 且任务明显与其相关，应先读取交接快照，再读取其中引用的 dev log、文件路径和当前代码。交接快照是当前状态提示，不替代代码和测试。

## 相关日志搜索关键词

优先使用：

- 用户明确提到的功能名、模块名、错误信息。
- 当前任务涉及的文件路径。
- git diff 中出现的文件名。
- 测试名称、命令名、接口名。
- 最近日志中的 `Next Steps` 和 `Open Questions`。

## 读取限制

默认限制：

- 最近日志最多 8 条。
- 相关历史日志最多 5 条。
- 总字符数粗略控制在 12000 以内。

超过限制时，优先读取：

1. `blocked` 或 `in_progress` 条目。
2. 标题或文件路径命中当前任务的条目。
3. 最近的 `done` 条目。
4. 包含 `Next Steps` 或 `Open Questions` 的条目。

## 写入流程

默认写入到：

```text
docs/dev-log/YYYY-MM-DD.md
```

如果当天文件不存在，创建：

```md
# Dev Log - YYYY-MM-DD
```

然后按时间追加条目。

## 条目结构

推荐结构：

```md
## YYYY-MM-DD HH:mm TZ - 标题

Agent: Codex | Claude Code | OpenCode | Other
Status: in_progress | done | blocked

### Summary
...

### Decisions
- ...

### Changes
- `path/to/file`: ...

### Verification
- `command`: passed | failed | not run

### Next Steps
- ...
```

根据实际情况省略空章节。不要为了填模板而制造空内容。

## 去重

写入前比较当天最近几条，默认 5 条。判断标准是事实是否等价，而不是文字是否完全一致。

重复时：

- 不追加新条目。
- 如果用户明确要求记录，简短说明已有等价记录。

部分重复但有新增事实时：

- 追加 update 条目。
- 只记录新增事实。

## 更新

默认不编辑旧条目。需要补充时追加 update：

```md
## YYYY-MM-DD HH:mm TZ - Update: 标题

Agent: Codex
Status: done

### Summary
补充此前记录缺失的信息。

### Decisions
- 新增或调整的决策。
```

## 纠错

旧记录有误时追加 correction：

```md
## YYYY-MM-DD HH:mm TZ - Correction: 标题

Agent: Codex
Status: done

### Summary
更正此前记录。

### Correction
- 旧记录：...
- 当前事实：...
- 验证来源：...
```

只有用户明确要求整理、重写、清理日志时，才可以编辑旧日志。

## 任务完成闭环

中等以上复杂度任务完成时，日志应包含：

- 做了什么。
- 为什么这么做。
- 改了哪里。
- 怎么验证。
- 还有什么没做。

完成条目控制在 20 到 40 行，不记录无关命令输出。

## Agent 交接快照

交接快照用于 agent 切换、长时间暂停或上下文可能丢失时，把当前工作台状态提炼给下一个 agent。它不是长期开发日志，不追加到 `docs/dev-log`。

默认写入：

```text
.agents/handoff/current.md
```

### 触发场景

用户明确表达以下意图时生成交接快照：

- “我要切换到 Claude/Codex/OpenCode”。
- “帮我生成 handoff”。
- “整理当前开发状态，让下一个 agent 接着做”。
- “暂停一下，把现在做到哪写下来”。
- “把当前上下文提炼成一份临时状态文档”。

不要把普通任务完成日志自动写成交接快照。除非用户明确要求，或当前上下文即将中断且交接价值很高。

### 写入前读取

生成交接快照前，优先收集：

1. 用户当前目标和边界要求。
2. 当前 git 状态和 diff 摘要。
3. 已读、已改、需要继续关注的文件路径。
4. 最近相关 dev log。
5. 本轮验证命令和结果。
6. 未解决问题、风险和下一步。

只写结论和接力信息，不复制聊天全文，不粘贴大段 diff 或命令输出。

### 文件生命周期

默认行为：

- `current.md` 表示当前临时状态，可以覆盖。
- 默认不提交到 git。
- 如果项目没有忽略 `.agents/handoff/`，可以提醒用户加入 `.gitignore`，但不要在未获授权时擅自修改。
- 用户要求保留历史时，再写入带时间戳的文件，例如 `.agents/handoff/2026-05-12-1530-codex-to-claude.md`。

### 推荐结构

```md
# Agent Handoff - 当前开发状态

Updated: YYYY-MM-DD HH:mm TZ
From Agent: Codex | Claude Code | OpenCode | Other
Suggested Next Agent: Claude Code | Codex | OpenCode | Other | Unknown
Status: in_progress | blocked | ready_for_next

## Current Goal
...

## Progress
...

## Current Context
...

## Decisions So Far
- ...

## Changed Files
- `path/to/file`: ...

## Verification
- `command`: passed | failed | not run

## Open Questions
- ...

## Next Steps
1. ...
2. ...

## Relevant Dev Logs
- `docs/dev-log/YYYY-MM-DD.md`: ...
```

根据实际情况省略空章节。优先让下一位 agent 能直接行动，而不是完整复述历史。

### 接手时读取

当 agent 读取交接快照接手任务时：

1. 先确认 `Updated` 时间，判断是否可能过期。
2. 读取 `Current Goal`、`Current Context` 和 `Next Steps`。
3. 打开 `Changed Files` 中列出的当前文件。
4. 读取 `Relevant Dev Logs` 中引用的日志。
5. 用当前代码、git diff 和测试结果验证快照内容。
6. 接手后若目标或状态明显变化，按用户要求更新 `current.md` 或写 dev log。
