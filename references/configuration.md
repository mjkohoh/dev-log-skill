# 配置说明

开发日志 skill 优先读取项目根目录的 `.devlog.yml`，其次读取 `docs/dev-log/config.yml`。如果没有配置文件，使用默认值。未知字段应忽略，非法字段应回退默认值。

## 推荐最小配置

```yaml
version: 1

log:
  dir: docs/dev-log
  audience: hybrid
  intended_for_git: true

handoff:
  enabled: true
  dir: .agents/handoff
  current_file: current.md
  intended_for_git: false

trigger:
  policy: conservative

privacy:
  redact_secrets: true
  redact_personal_data: true
  allow_sensitive_content: false
```

## 完整配置

```yaml
version: 1

log:
  dir: docs/dev-log
  file_mode: daily # daily | single | session
  language: zh-CN
  timezone: local
  audience: hybrid # human | agent | hybrid
  intended_for_git: true

trigger:
  policy: conservative # explicit_only | conservative | balanced | eager
  min_value: meaningful # meaningful | minor
  record_on_user_request: true
  record_task_completion: true
  record_blockers: true
  record_decisions: true

read:
  on_task_start: true
  recent_days: 3
  max_entries: 8
  search_related: true
  max_related_entries: 5
  max_total_chars: 12000
  trust_level: advisory # advisory | strong
  summarize_before_use: true

write:
  append_only: true
  edit_previous_entries: false
  create_file_if_missing: true
  entry_order: chronological
  max_entry_lines: 40

handoff:
  enabled: true
  dir: .agents/handoff
  current_file: current.md
  intended_for_git: false
  overwrite_current: true
  include_git_diff_summary: true
  include_recent_dev_logs: true
  create_dir_if_missing: true
  timestamped_history: false

dedupe:
  enabled: true
  compare_recent_entries: 5
  prefer_update_over_duplicate: true

privacy:
  redact_secrets: true
  redact_personal_data: true
  allow_sensitive_content: false
  secret_placeholders:
    - "***"
    - "[REDACTED]"

index:
  enabled: false
  path: docs/dev-log/INDEX.md
```

## 字段说明

### `log`

- `dir`: 日志目录。默认 `docs/dev-log`。
- `file_mode`: 日志文件组织方式。第一版推荐使用 `daily`。
- `language`: 日志语言。默认中文优先。
- `timezone`: 时间戳时区。`local` 表示使用当前环境时区。
- `audience`: 日志风格。`human` 偏叙述，`agent` 偏结构化，`hybrid` 兼顾两者。
- `intended_for_git`: 是否默认可以提交到 git。为 `true` 时必须执行严格脱敏。

### `handoff`

- `enabled`: 是否启用 agent 交接快照能力。
- `dir`: 交接快照目录。默认 `.agents/handoff`。
- `current_file`: 当前临时状态文件名。默认 `current.md`。
- `intended_for_git`: 交接快照是否默认可以提交到 git。默认 `false`。
- `overwrite_current`: 是否允许覆盖当前状态文件。默认 `true`。
- `include_git_diff_summary`: 生成快照时是否包含 git diff 摘要。
- `include_recent_dev_logs`: 生成快照时是否引用相关 dev log。
- `create_dir_if_missing`: 目录不存在时是否创建。
- `timestamped_history`: 是否默认写入带时间戳的历史快照。默认 `false`，只维护 `current.md`。

### `trigger`

- `policy`: 自动记录积极程度。
- `min_value`: 自动记录的最低价值门槛。默认只有有接力价值的信息才记录。
- `record_on_user_request`: 用户明确要求时是否尝试记录。
- `record_task_completion`: 中等以上任务完成时是否记录。
- `record_blockers`: 阻塞、根因、重要限制是否记录。
- `record_decisions`: 关键决策是否记录。

### `read`

- `on_task_start`: 复杂开发任务开始前是否读取日志。
- `recent_days`: 默认读取最近几天。
- `max_entries`: 最近日志最多读取条目数。
- `search_related`: 是否搜索相关历史日志。
- `max_related_entries`: 相关历史日志最多读取条目数。
- `max_total_chars`: 读取日志内容的粗略上限。
- `trust_level`: 日志可信度。默认 `advisory`，表示仅作参考。
- `summarize_before_use`: 是否先总结再使用日志上下文。

### `write`

- `append_only`: 是否只追加新条目。
- `edit_previous_entries`: 是否允许主动编辑旧条目。默认不允许。
- `create_file_if_missing`: 当天日志文件不存在时是否创建。
- `entry_order`: 条目顺序。默认按时间追加。
- `max_entry_lines`: 单条日志建议最大行数。

### `dedupe`

- `enabled`: 是否启用去重。
- `compare_recent_entries`: 写入前比较最近几条。
- `prefer_update_over_duplicate`: 发现部分重复但有新增事实时，追加 update 而不是完整重复条目。

### `privacy`

- `redact_secrets`: 是否脱敏密钥和凭证。
- `redact_personal_data`: 是否脱敏个人信息。
- `allow_sensitive_content`: 是否允许写入敏感内容。默认必须为 `false`。
- `secret_placeholders`: 脱敏占位符。

### `index`

- `enabled`: 是否维护索引文件。第一版默认关闭。
- `path`: 索引文件路径。
