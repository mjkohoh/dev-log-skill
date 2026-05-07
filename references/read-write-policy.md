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
