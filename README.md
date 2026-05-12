# Dev Log Skill

Dev Log Skill 是一个通用 AI coding agent 开发日志协议。它帮助 Codex、Claude Code、OpenCode 以及其他 agent 在开发过程中按需记录 Markdown 日志，保存关键决策、阻塞原因、验证结果和接力信息。

默认日志写入当前项目：

```text
docs/dev-log/YYYY-MM-DD.md
```

日志默认可以提交到 git，因此这个 skill 明确要求去重、克制记录和敏感数据脱敏。

这个 skill 也提供临时 agent 交接快照能力：当你从 Codex 切换到 Claude Code、OpenCode 或其他 agent 时，可以生成 `.agents/handoff/current.md`，让下一个 agent 先读取当前目标、进度、约束和下一步。

## 适用场景

- 你希望 agent 在开发代码时留下可读的开发日志。
- 你希望后续 agent 可以通过日志理解之前的决策和上下文。
- 你希望在明确说“记录一下日志”时，agent 能判断价值、去重并写入日志。
- 你希望开发日志是项目文档的一部分，可以随着代码一起版本化。
- 你希望切换 agent 前生成一份临时状态文档，让下一个 agent 直接接着做。

## 包结构

```text
dev-log-skill/
├── SKILL.md
├── README.md
├── AGENTS.md
├── templates/
│   ├── DEV_LOG_ENTRY.md
│   ├── HANDOFF.md
│   └── devlog.config.yml
├── references/
│   ├── configuration.md
│   ├── trigger-policy.md
│   ├── read-write-policy.md
│   └── privacy-redaction.md
├── examples/
│   ├── hybrid-log.md
│   ├── human-focused-log.md
│   ├── agent-focused-log.md
│   └── handoff-current.md
└── evals/
    └── evals.json
```

## 安装到项目

这个仓库是 skill 源包。使用时，将整个目录复制或同步到目标项目的 agent skill 目录。

### Claude Code

项目级 skill 推荐位置：

```text
.claude/skills/dev-log/
```

目录内应包含 `SKILL.md`：

```text
.claude/skills/dev-log/SKILL.md
```

### OpenCode

OpenCode 支持多个位置，项目级推荐：

```text
.opencode/skills/dev-log/
```

OpenCode 也兼容：

```text
.claude/skills/dev-log/
.agents/skills/dev-log/
```

### Codex 和其他 agent

如果 agent 支持 skills，按其 skill 管理方式安装本目录。如果 agent 没有专用 skill 机制，也可以把 `SKILL.md` 作为项目级 agent instruction 引入，并保留 `templates/`、`references/`、`examples/` 供 agent 按需读取。

## 项目配置

目标项目可以创建 `.devlog.yml` 来覆盖默认行为：

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

完整配置见 [references/configuration.md](references/configuration.md)。

## 如何使用

可以直接对 agent 说：

```text
记录一下刚才的开发日志。
```

```text
继续上次任务，先看 dev log。
```

```text
这次实现完成后，把关键决策和验证结果写入开发日志。
```

```text
这次调试过程有点绕，结束时帮我写一条开发日志，注意脱敏。
```

```text
我要切换到 Claude，帮我生成一份 handoff。
```

```text
继续上次任务，先读 .agents/handoff/current.md。
```

默认策略比较克制。agent 不应该把所有操作都写成流水账，而是记录能帮助未来的人或 agent 接力的事实。

## 交接快照

交接快照不是开发日志。它是临时的当前状态文档，默认写入：

```text
.agents/handoff/current.md
```

建议把 `.agents/handoff/` 加入目标项目的 `.gitignore`，除非团队明确希望提交交接状态。默认规则允许覆盖 `current.md`，因为它代表“现在做到哪了”。

交接快照通常包含：

- 当前开发目标。
- 已完成进度和当前卡点。
- 用户要求、边界条件和重要约束。
- 已做出的关键决策。
- 当前改动文件和注意事项。
- 验证命令与结果。
- 下一个 agent 应该执行的步骤。
- 相关 dev log 引用。

示例见 [examples/handoff-current.md](examples/handoff-current.md)，模板见 [templates/HANDOFF.md](templates/HANDOFF.md)。

## 默认日志格式

默认使用 hybrid 风格：

```md
## 2026-05-07 14:10 CST - 登录限流改为 Redis 计数

Agent: Codex
Status: done

### Summary
登录失败限流从进程内计数调整为 Redis 计数，因为服务存在多实例部署，内存状态无法跨实例共享。

### Decisions
- 使用 Redis 保存登录失败次数。
- key 格式为 `login:fail:{user_id}`。
- 失败计数 TTL 为 15 分钟。

### Changes
- `src/auth/login.ts`: 接入失败计数和成功清理逻辑。

### Verification
- `npm test -- tests/auth/login.test.ts`: passed

### Next Steps
- 如果后续增加 IP 维度限流，需要单独设计 key 命名和误伤策略。
```

更多示例见 `examples/`。

## `evals` 是什么

`evals/` 不是使用 skill 的必需部分，也不是 Codex 专用运行文件。它是这个 skill 的验收样例，用来测试未来修改是否破坏核心行为。

它主要验证：

- 用户主动要求记录时是否会写日志。
- 已有等价记录时是否避免重复。
- 有新增事实时是否追加 update。
- 任务完成时是否形成可接力闭环。
- 包含敏感信息时是否正确脱敏。
- 继续任务时是否先读取相关日志。
- 切换 agent 前是否生成临时交接快照。
- 接手任务时是否优先读取 handoff 并验证当前代码。

不同 agent 不一定会自动消费 `evals/evals.json`。需要测试时，可以让 agent 读取这些样例并模拟执行。

## 隐私原则

日志默认可以提交到 git。不要原样记录：

- API key、token、cookie、session、JWT。
- 密码、私钥、证书内容。
- 真实手机号、身份证号、银行卡号。
- 生产用户原始数据。
- 带签名参数的 URL。
- `.env` 和生产配置里的敏感值。

更多规则见 [references/privacy-redaction.md](references/privacy-redaction.md)。

交接快照虽然默认不提交，也应执行同样的脱敏规则。它可能更接近当前工作台状态，更容易混入临时凭证、原始响应或用户数据，因此宁可少写，也不要写入未经脱敏的敏感信息。

## 设计取向

这个 skill 故意不依赖脚本。第一版以 Markdown 指令、模板、参考资料和示例为核心，保证可以被不同 agent 使用。后续如果真实使用中发现日期文件创建、查重或脱敏成本很高，可以再增加可选脚本，但脚本不应成为基本使用门槛。
