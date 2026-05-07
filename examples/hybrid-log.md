# Dev Log - 2026-05-07

## 2026-05-07 14:10 CST - 登录限流改为 Redis 计数

Agent: Codex
Status: done

### Summary
登录失败限流从进程内计数调整为 Redis 计数，因为服务存在多实例部署，内存状态无法跨实例共享。

### Decisions
- 使用 Redis 保存登录失败次数。
- key 格式为 `login:fail:{user_id}`。
- 失败计数 TTL 为 15 分钟。
- 登录成功后清理失败计数。

### Changes
- `src/auth/login.ts`: 接入失败计数和成功清理逻辑。
- `src/auth/rateLimit.ts`: 封装登录限流判断。
- `tests/auth/login.test.ts`: 覆盖失败累计、TTL、成功清理。

### Verification
- `npm test -- tests/auth/login.test.ts`: passed

### Next Steps
- 如果后续增加 IP 维度限流，需要单独设计 key 命名和误伤策略。
