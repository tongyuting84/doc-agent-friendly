---
spec_id: api-agent-invocation
spec_version: "2.0"
applies_to: [api, api_explorer, sdk, signed_http]
requires: [global-agent-execution, api-credential-configuration]
---

# API Agent 调用规范

> 适用于 Agent 根据公开 API 文档，使用 API Explorer、官方 SDK 或签名 HTTP 调用单个 OpenAPI。通用安全、批准和验收规则由[全局 Agent 执行规范](global-agent-execution.md)定义；认证准备由[Agent API 凭证配置指南](api-credential-configuration.md)定义。

## 1. 规范边界

本规范负责提取 API 契约、校验请求参数、选择 API 执行渠道、解析 Endpoint、发送请求以及验证响应、分页和异步结果。它不重复定义跨 API 的业务流程，也不把 CLI 或 Kubernetes YAML 当作 API 执行渠道。

当前 API 文档中的接口事实优先于通用默认值，服务端实际校验和返回结果优先于文档示例。

## 2. 标准调用流程

```text
读取当前 API 文档
→ 提取 API 契约和操作语义
→ 获取并校验业务输入
→ 选择执行渠道
→ 按凭证指南检查认证
→ 解析并确认 Endpoint
→ 生成脱敏请求摘要
→ 发送请求
→ 验证传输层、API 层和结果层
→ 处理分页或异步终态
→ 输出结构化结果
```

## 3. API 契约

发送前必须从 API 文档提取：

```yaml
api_contract:
  product: <product>
  action: <action>
  version: <api-version>
  method: <http-method>
  service: <service-name>
  region: <target-region-or-not-applicable>
  endpoint: <documented-endpoint>
  required_parameters: []
  optional_parameters: []
  parameter_constraints: []
  response_fields: []
  error_codes: []
  operation_type: query | create | update | delete | action
  execution_mode: synchronous | asynchronous
  idempotency: documented | token_supported | undefined
```

缺少发送请求所必需的事实，或无法确定写操作的副作用、异步行为和验证方式时，返回 `safe_blocked`，不得按名称猜测。

## 4. 输入校验

- 必填参数来自用户确认、可信上下文、现有资源或文档指定的只读查询。
- 校验类型、长度、枚举、范围、数组数量、时间格式、条件依赖、地域和资源归属。
- 文档明确给出的默认值可以使用，但输出中应说明；没有默认值时不得自行补充。
- 示例值、脱敏值和占位符不能用于真实请求。
- 多个有效资源无法唯一选择时必须询问用户。
- 敏感参数不得进入请求摘要和日志。

## 5. 执行渠道

```yaml
execution_method:
  allowed_values: [auto, api_explorer, sdk, signed_http]
  default: auto
```

用户指定渠道时优先遵守；不可用时不得静默切换。`auto` 可以根据登录态、SDK 支持、凭证、网络和 Action/Version 支持情况选择渠道，并记录理由。

- API Explorer 使用浏览器登录会话。
- SDK 使用官方支持的凭证链及版本。
- 签名 HTTP 使用受支持的签名方式和安全凭证。
- 三种渠道的认证环境相互独立，具体规则见凭证指南。
- 渠道切换不得规避认证失败、权限不足或业务参数错误。

## 6. Endpoint 解析

Endpoint 按以下顺序确定：当前 API 文档、产品服务地址文档、API 文档明确链接的调用方式文档、官方 SDK 的有效映射。SDK 示例或默认地址不得覆盖更高优先级地址。

使用 SDK 时比较文档地址和实际地址；不一致且支持自定义时显式设置为文档推荐地址。无法使用推荐地址时，`auto` 可切换渠道；用户指定 SDK 时返回 `safe_blocked`。

## 7. 发送前检查

```yaml
api_preflight:
  contract_complete: true
  required_parameters_complete: true
  examples_used_as_real_values: false
  execution_method: api_explorer | sdk | signed_http
  authentication_ready: true
  endpoint_verified: true
  write_approval_ready: true | false | not_applicable
  secrets_logged: false
```

发送前展示脱敏摘要，至少包含 Action、Version、Region、HTTP 方法、实际 Endpoint、Endpoint 来源、渠道和参数。写操作还必须满足全局确认要求。

## 8. 响应验证

必须验证三个层次：

1. **传输层**：响应、HTTP、网络、TLS、超时和序列化。
2. **API 层**：错误码、RequestId、响应结构及 Action/Version/Service/Region。
3. **结果层**：字段类型、筛选条件、资源 ID、状态和文档声明的成功条件。

创建、更新、删除或操作接口必须重新查询真实状态；只返回 ID 或“已受理”不能证明资源可用。

## 9. 查询、分页与空结果

- 空列表通常表示请求成功但没有匹配数据，应分别报告 `request_succeeded: true` 和 `match_found: false`。
- 用户要求完整结果时获取全部页面，遵守页面限制并防止游标循环。
- 任一页面失败时不得把已有部分标记为完整结果。
- 输出实际页数和条目数。

## 10. 异步与写操作

异步接口必须提取任务/资源/操作 ID、状态查询 API、进行中/成功/失败状态、轮询间隔和观察窗口。缺少状态查询方式或成功定义时返回 `safe_blocked`。

写请求超时后不得立即重发；先使用幂等令牌、资源名称、任务 ID 或查询接口确认结果。只有文档保证幂等时才可有限重试，无法确定时安全停止并保留证据。

## 11. 错误与重试

优先按专有错误码、公共错误码、HTTP 状态和 SDK 结构化错误分类。限流、临时网络或服务错误可以有限退避重试；认证、权限、参数、配额、库存、资源冲突和非幂等写结果不确定不得自动重试。

认证失败按凭证指南处理；权限拒绝不得通过切换身份或渠道规避。未知错误保持为未知，并保存脱敏错误和 RequestId。

## 12. 输出要求

```yaml
status: verified_success | safe_blocked | failure
execution:
  requested_method: auto
  actual_method: sdk
  endpoint: <endpoint>
  endpoint_source: <source>
api:
  product: <product>
  action: <action>
  version: <version>
  region: <region-or-null>
response:
  request_id: <request-id>
  resource_id: <resource-id-or-null>
  task_id: <task-id-or-null>
verification:
  transport_passed: true
  api_passed: true
  result_passed: true
retry_count: 0
warnings: []
```

不得输出凭证、签名、令牌、Cookie、密码或未脱敏数据。

## 13. API 文档引用方式

API 页面只需引用全局规范，不必分别链接本规范和凭证指南：

```markdown
> AI Agent 执行入口：执行本接口前，请读取并遵守[《全局 Agent 执行规范》](<公开地址>)。
```

全局规范会将 API 操作强制路由到本规范和凭证指南。无法从正文可靠表达的接口特殊语义，可以在入口后用不超过三句补充；不得为每个 API 创建重复规范。
