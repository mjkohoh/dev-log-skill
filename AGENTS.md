# AGENTS.md

## 项目定位

本项目是一个通用 AI coding agent 的开发日志 skill 源包。目标用户包括 Codex、Claude Code、OpenCode，以及其他可以读取 `SKILL.md` 的 agent。

## 语言和风格

- 面向人类的文档、模板、示例默认使用中文。
- 保持 agent-agnostic，不要把规则写成某个单一平台专用。
- 优先使用 Markdown、YAML 和纯文本协议，避免引入非必要脚本依赖。

## 开发约定

- `SKILL.md` 保持精炼，放 agent 每次触发时必须知道的操作规程。
- 详细策略放入 `references/`。
- 可复用模板放入 `templates/`。
- 风格样例放入 `examples/`。
- 验收样例放入 `evals/`，它们用于测试 skill 行为，不是运行 skill 的必需部分。

## 质量检查

修改后至少检查：

- `SKILL.md` frontmatter 包含合法的 `name` 和 `description`。
- README 说明如何在 Codex、Claude Code、OpenCode 或通用 agent 中使用。
- 脱敏规则没有放松。
- 示例日志不会误导 agent 写流水账或记录敏感信息。
