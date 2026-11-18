# 通用 Agent 规则

## 唯一入口

产品文档原则上只引用[全局 Agent 执行规范](01-global-agent-execution.md)。全局规范识别文档中的实际操作类型，并强制加载对应子规范。

| 规范 | 职责 | 加载条件 |
| --- | --- | --- |
| [全局 Agent 执行规范](01-global-agent-execution.md) | 通用流程、输入、安全、风险、验收及规范路由 | 所有 Agent 可执行文档 |
| [API Agent 调用规范](02-api-agent-invocation.md) | API 契约、渠道、Endpoint、请求和响应验证 | API、SDK、API Explorer、签名 HTTP |
| [Agent API 凭证配置指南](03-api-credential-configuration.md) | API 认证来源、隔离、生命周期和脱敏 | 与 API 规范一同加载 |
| [控制台 Agent 操作规范](04-console-agent-operation.md) | 页面字段、表单、浏览器会话、控制台验收 | 控制台页面、按钮、表单或向导 |
| [Kubernetes YAML Agent 执行规范](05-kubernetes-yaml-agent-execution.md) | kubectl、kubeconfig、集群连接、YAML 校验和验收 | Kubernetes YAML 或 kubectl |

同一文档包含多种操作时，应加载所有适用子规范。子规范不能脱离全局规范单独降低安全或确认要求。

## 维护说明

最初四份规则于 2026-11-16 从 `https://code.byted.org/tongyuting/doc-use` 迁入。2026-11-17 起按“全局入口 + 操作子规范”架构维护，并新增 Kubernetes YAML 执行规范。`main` 分支是唯一事实来源。
