# 通用 Agent 规则

| 规则 | 使用场景 |
|---|---|
| [全局 Agent 执行规范](global-agent-execution.md) | 所有 API、SDK、CLI、控制台和任务文档 |
| [通用 API Agent 调用规范](api-agent-invocation.md) | 调用单个 OpenAPI |
| [Agent API 凭证配置指南](api-credential-configuration.md) | SDK、CLI 或签名 HTTP 需要认证时 |
| [控制台任务 Agent 调用规范](console-task-agent-invocation.md) | 通过云控制台完成任务时 |

## 迁移说明

以上四份规则于 2026-11-16 从 `https://code.byted.org/tongyuting/doc-use` 的 `master` 分支迁移并重命名。迁移时未修改正文语义；仅将规则之间的内部 Codebase 链接改为本仓库相对链接。

后续以本仓库 `main` 分支为维护入口，避免两个仓库双向人工编辑造成规则漂移。
