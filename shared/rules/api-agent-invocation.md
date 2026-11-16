# 通用 API Agent 调用规范

规范版本：v1.1 

> 适用范围：通过公开 API 文档调用火山引擎 OpenAPI 的 AI Agent 

本文定义 AI Agent 如何读取一篇 API 文档、构造一次 API 请求、处理响应，并判断本次 API 调用是否完成。

本文不定义账号身份、凭证管理、授权审批、风险治理、审计留痕和资源清理的通用要求。这些要求由 
[全局 Agent 执行规范](global-agent-execution.md)
 统一定义。

## 1. 规范边界

### 1.1 本规范负责

- 从当前 API 文档提取调用契约；

- 确认接口类型和调用模式；

- 检查接口参数；

- 选择 API 文档明确支持的调用方式；

- 构造和发送请求；

- 验证响应元数据和结果字段；

- 处理分页、异步状态、空结果和 API 错误；

- 对可重试错误执行有限重试；

- 输出结构化的 API 调用结果。

### 1.2 本规范不负责

- 判断用户是否有权执行某项业务变更；

- 代替 IAM、服务端校验和审批系统；

- 定义跨多个 API 的业务任务流程；

- 为每个 API 重复编写请求参数和返回字段；

- 推断 API 文档没有声明的业务成功条件；

- 决定资源创建后的清理策略；

- 绕过登录、权限、配额、费用或安全限制。

## 2. 文档关系

Agent 调用单个 API 时使用两类文档：

- 全局 Agent 执行规范：负责 身份、权限、安全、批准、审计和通用停止条件。

- 当前 API 文档：负责 Action、Version、地址、参数、返回字段、限制和错误码。

- 通用 API Agent 调用规范：负责如何读取 API 文档、发起调用和验证响应。

当前 API 文档中的接口事实优先于本文中的通用默认值。服务端实际校验和返回结果优先于所有文档示例。

## 3. 标准调用流程

Agent 必须按照以下顺序执行：

```
读取 API 文档
→ 提取调用契约
→ 判断接口类型
→ 检查输入
→ 检查全局执行条件
→ 构造请求
→ 发起调用
→ 验证响应
→ 处理分页或异步状态
→ 输出结构化结果
```

任何必需信息不明确时，不得跳过或猜测。

## 4. 提取调用契约

发送请求前，Agent 必须从当前 API 文档中提取以下信息：

```yaml
api_contract:
  product: <product>
  action: <action>
  version: <api-version>
  method: <http-method>
  service: <service-name>
  endpoint: <regional-or-global-endpoint>
  endpoint_source: <official-document-or-sdk-resolution>
  region: <target-region-or-not-applicable>
  required_parameters: []
  optional_parameters: []
  parameter_constraints: []
  response_fields: []
  error_codes: []
```

至少确认：

1. API Action；

1. API Version；

1. HTTP 请求方法；

1. 服务地址及地域规则；

1. 必填参数；

1. 参数类型、数量、格式、默认值和依赖关系；

1. 公共参数要求；

1. 返回结果的根字段；

1. 专有错误码和公共错误码入口。

如果 API 文档缺少发送请求所必需的信息，应返回 safe_blocked，并指出缺失字段。

## 5. 判断接口类型

Agent 应根据 API 文档对接口进行分类，而不是仅根据 Action 名称推断。

### 5.1 操作类型

| 类型 | 判断依据 | 调用后要求 |
| --- | --- | --- |
| 查询 | 文档说明只读取资源或状态 | 验证响应结构和查询语义 |
| 创建 | 文档说明创建资源、任务或配置 | 保存资源 ID，并验证创建结果 |
| 更新 | 文档说明修改现有资源或配置 | 重新读取资源并验证变更 |
| 删除 | 文档说明删除、释放或解绑资源 | 查询确认目标已进入文档定义的删除终态 |
| 操作 | 文档说明启动、停止、重启、执行命令等 | 验证动作产生的真实状态变化 |

### 5.2 同步与异步

- 同步接口：响应中直接提供本次调用的最终 API 结果。

- 异步接口：响应只表示请求已受理，并返回任务 ID、资源 ID或操作 ID；Agent 必须按照 API 文档查询终态。

如果文档没有明确接口是否异步，但响应只包含 ID 或受理状态，Agent 不得自行宣布业务操作完成，应返回 safe_blocked 或继续查找文档明确指定的查询方式。

### 5.3 幂等属性

仅在 API 文档满足以下任一条件时，将写操作视为幂等：

- 文档明确声明接口幂等；

- 文档提供 ClientToken、IdempotencyKey 等幂等参数；

- 文档明确说明重复请求行为。

未声明幂等性的写接口不得因为超时而直接重复发送。

### 5.4 无法确定类型

如果无法可靠判断接口是否有副作用、是否异步或是否可重试，Agent 必须停止，并输出：

```json
{
  "status": "safe_blocked",
  "blocker": "operation_semantics_undefined",
  "required_documentation": [
    "side_effects",
    "async_behavior",
    "verification_method"
  ]
}
```

## 6. 输入检查

### 6.1 执行输入获取规则

Agent 应按照以下顺序获取接口必选参数：

1. 使用用户在当前请求中明确提供的值。

1. 使用当前对话或上游任务已经确认的值。

1. 从用户已授权访问的云资源中查询确定值。

1. 根据文档中明确给出的参数来源，通过只读接口获取。

1. 如果存在多个有效候选项且无法唯一确定，向用户确认。

1. 如果无法获得真实值，停止执行。

Agent 不得：

- 使用文档示例值代替真实业务参数。

- 猜测地域、资源 ID、实例规格或资源名称。

- 在多个候选资源中擅自选择。

- 为避免询问用户而虚构参数。

只有当缺少的信息会影响调用结果或资源范围时，才中断并询问用户。

### 6.2 必填参数

- 所有必填参数必须有值；

- 值必须来自用户、可信配置、现有资源或文档明确指定的查询结果；

- 不得使用示例值、脱敏值和占位符执行真实请求；

- 不得猜测资源 ID、地域、版本、计费方式和权限。

### 6.3 类型与范围

Agent 必须检查：

- 字符串、布尔值、整数、数组和对象类型；

- 数值范围；

- 数组元素数量；

- 枚举值；

- 字符长度；

- 时间和日期格式；

- 参数之间的条件依赖；

- 地域、可用区和资源归属关系。

### 6.4 默认值

- 文档明确给出默认值时，可以省略对应参数；

- 最终输出应说明使用了文档默认值；

- 文档没有默认值时，不得自行设置业务默认值。

### 6.5 敏感参数

敏感参数的获取、传输和输出遵守 
[全局 Agent 执行规范](global-agent-execution.md)
 。API 请求日志中只能保留脱敏摘要。

### 6.6 执行条件分类

Agent 应区分：

| 类型 | 示例 | 获取规则 |
| --- | --- | --- |
| 接口契约 | Action、Version、Service、方法 | 从官方文档直接提取，不询问用户 |
| 业务输入 | Region、资源 ID、筛选条件 | 从用户请求、上下文、现有资源或文档指定的只读查询获取 |
| 执行环境 | 凭证、SDK、CLI、网络、浏览器登录态 | 检测当前环境；缺失时引导用户安全准备 |

Agent 应先自动获取可以确定的信息，只在缺失值影响结果或资源范围且无法唯一确定时打断用户。

## 7. 检查执行条件

发送请求前，Agent 必须按照 
[全局 Agent 执行规范](global-agent-execution.md)
 确认：

- 当前身份和权限满足要求；

- 有副作用的操作已完成必要的计划和批准；

- 费用、配额和影响范围已经明确；

- 未触发全局停止条件。

本规范不重复定义这些规则，也不降低全局规范的要求。

### 7.1 标准执行前检查

~~~yaml
preflight:
  api_contract:
    action: resolved
    version: resolved
    service: resolved
    method: resolved
    region: resolved
    endpoint: resolved
  business_inputs:
    required_parameters: complete
    examples_used_as_real_values: false
  runtime:
    execution_method: resolved
    authentication_available: true
    network_available: true
  sdk:
    package_installed: true
    endpoint_verified: true
    credentials_not_hardcoded: true
~~~

不适用的检查项可以标记为 not_applicable；其余必需项未满足时不得发送请求。

## 8. 选择执行方式

API 参数决定调用什么，执行方式决定通过什么工具发送请求。执行方式属于调用执行参数，不属于服务端业务参数。

### 8.1 执行方式参数

~~~yaml
execution_method:
  type: enum
  allowed_values:
    - auto
    - api_explorer
    - sdk
    - cli
    - signed_http
  default: auto
~~~

用户可以通过自然语言指定执行方式。用户明确指定时，Agent 必须优先遵守；指定方式不可用时不得静默切换，应说明缺少的条件和可用替代方式，并获得用户允许后再切换。

### 8.2 自动选择

用户未指定时使用 auto。Agent 应检查 API Explorer 登录状态、SDK 和 CLI 是否安装、凭证是否可用、工具是否支持当前 Action 和 Version、签名客户端是否已正确实现，以及网络是否可用。

交互式场景建议优先使用 API Explorer、官方 SDK、官方 CLI、签名 HTTP；自动化场景建议优先使用官方 SDK、官方 CLI、签名 HTTP、API Explorer。只有 auto 模式可以自动切换方式。

确定方式后，应输出实际方式和选择原因。只读无副作用接口可以在输出摘要后直接执行；写操作仍须遵守全局批准规则。

### 8.3 认证环境隔离

API Explorer、SDK、CLI 和签名 HTTP 的认证状态相互独立。浏览器已登录不代表 SDK 或 CLI 已认证。Agent 不得导出浏览器 Cookie、登录凭证或临时令牌供其他客户端使用。凭证准备遵守《全局 Agent 执行规范》和[《Agent API 凭证配置指南》](api-credential-configuration.md)。

### 8.4 执行方式切换

- auto 模式下，当前方式不可用或不支持目标接口时，可以切换到下一个可用方式，并记录原因；
- 用户指定方式时，切换前必须获得用户允许；
- 认证失败、权限不足和业务参数错误不得通过切换方式规避；
- API Explorer 和 SDK 的结果都必须经过相同的响应与结果验证。

### 8.5 SDK 使用要求

使用官方 SDK 时，Agent 必须：

- 优先安装当前产品所需的最小 SDK 包，避免无必要地安装全部产品聚合包；
- 使用受信任凭证链、Profile、工作负载身份或安全环境变量，不把 AK/SK 写入代码；
- 保存并验证 SDK 返回对象，不得使用空 pass 吞掉异常；
- 确认 SDK 版本支持当前 Action 和 Version；
- 验证 SDK 实际使用的 Region、Service 和 Endpoint；
- 不把 SDK 生成示例中的占位符、默认地址和静态示例当作接口事实。

## 8.6 Endpoint 解析与冲突处理

Endpoint 按以下优先级确定：

1. 当前 API 文档明确指定的服务地址；
2. 当前产品“服务地址”文档中与目标 Region 对应的地址；
3. 当前 API 文档明确链接的产品调用方式文档；
4. 官方 SDK 对当前 Service 和 Region 的有效 Endpoint 映射。

API Explorer、SDK、CLI 生成示例中的默认地址不得覆盖 API 文档或产品服务地址文档明确推荐的地址。示例地址只有在目标 Region 一致、文档声明仍有效且没有更高优先级规则时才可使用。

使用 SDK 时必须比较 documented_endpoint 与 sdk_endpoint：

~~~yaml
endpoint_check:
  service: <service>
  region: <region>
  documented_endpoint: <official-endpoint>
  sdk_endpoint: <actual-sdk-endpoint>
  matched: true
~~~

如果 SDK 默认地址不一致且支持自定义 Host 或 Endpoint，Agent 必须显式设置为文档推荐地址。如果 SDK 无法使用推荐地址，auto 模式应切换其他可用方式；用户指定 SDK 时返回 safe_blocked。不得仅以域名可访问或 SDK 默认值作为地址正确的证据。

发送前摘要必须包含实际 Endpoint、来源以及是否覆盖 SDK 默认值。

## 9. 构造请求

请求必须满足：

1. Action 与当前 API 文档一致；

1. Version 与当前 API 文档一致；

1. 请求方法和服务地址正确；

1. 地域与目标资源一致；

1. 公共参数完整；

1. 请求体或查询参数符合类型定义；

1. 条件必填参数已经根据实际场景补齐；

1. 幂等参数符合文档要求；

1. 日志中没有凭证、签名和敏感值。

发送前生成脱敏请求摘要：

```json
{
  "action": "<action>",
  "version": "<version>",
  "region": "<region>",
  "method": "<method>",
  "endpoint": "<endpoint>",
  "parameters": {
    "<parameter>": "<sanitized-value>"
  }
}
```

## 10. 验证响应

验证分为三个层次，不得只检查其中一个层次。

### 10.1 传输层

检查：

- 请求是否收到响应；

- HTTP 状态是否符合 API 文档；

- 客户端是否发生网络、TLS、超时或序列化错误。

### 10.2 API 层

检查：

- API 是否返回错误码；

- RequestId 是否存在；

- 返回的 Action、Version、Service 和 Region 是否与请求一致；

- 响应结构是否包含 API 文档声明的结果字段。

如果某个产品的公共响应字段名称不同，以其公共返回参数文档为准。

### 10.3 结果层

检查：

- 结果字段类型是否与文档一致；

- 请求中的资源或筛选条件是否与返回结果对应；

- 必需的资源 ID、状态或数据字段是否非空；

- 枚举状态是否属于文档声明范围；

- API 文档声明的特殊成功条件是否满足。

API 请求返回成功不必然代表业务目标完成。本规范只判断本次 API 调用是否完成；跨接口业务目标由独立任务规范判断。

## 11. 查询接口

### 11.1 查询成功

查询接口满足以下条件时，可以将调用标记为 verified_success：

- 请求和 API 层验证通过；

- 响应结果字段存在且类型正确；

- 查询条件与返回数据能够对应；

- 结果中的必需字段完整。

### 11.2 空结果

对于查询接口，空列表或空对象通常表示“请求执行成功，但没有匹配数据”，不应自动判定为调用失败。

Agent 必须分别输出：

```json
{
  "request_succeeded": true,
  "match_found": false
}
```

如果 API 文档对空结果另有定义，以该定义为准。

### 11.3 分页

如果 API 文档包含分页参数或分页令牌：

- 用户只要求一页时，按指定页执行；

- 用户要求完整结果时，继续获取后续页面；

- 遵守页面大小上限；

- 防止页码、游标或令牌循环；

- 记录实际获取的页数和条目数；

- 任一页面失败时，不得把不完整数据标记为完整结果。

## 12. 创建和更新接口

### 12.1 创建接口

- 保存创建请求的 RequestId；

- 保存服务端返回的资源 ID、任务 ID 或操作 ID；

- 文档声明异步时，调用指定查询接口验证终态；

- 文档提供幂等令牌时，同一执行意图的重试复用原令牌；

- 只返回 ID 不能证明资源已经可用。

### 12.2 更新接口

- 执行前保存目标资源的关键状态；

- 执行后重新读取资源；

- 比较实际状态与请求变更；

- 未观察到预期变化时，不得宣布更新成功；

- 发现非计划变化时停止，并按照全局规范处理。

## 13. 删除和操作接口

### 13.1 删除接口

- 只使用经过确认的精确资源 ID；

- 执行前遵守全局删除确认要求；

- 请求成功后查询资源或删除任务状态；

- 只有达到 API 文档定义的删除终态，才标记调用完成；

- 查询接口返回暂时不可见或延迟时，按照文档的最终一致性说明处理。

### 13.2 操作接口

对于启动、停止、重启、执行命令等接口：

- 保存操作 ID 或 RequestId；

- 按文档查询目标状态；

- 验证状态变化与请求一致；

- 接口受理成功不等于动作已经生效。

## 14. 异步接口

API 文档声明异步时，Agent 必须提取：

```yaml
async_contract:
  initial_response_id: <resource-id-task-id-or-operation-id>
  status_api: <documented-status-api>
  in_progress_states: []
  success_states: []
  failure_states: []
  polling_interval: <documented-or-conservative-value>
  observation_timeout: <documented-or-user-approved-value>
```

轮询规则：

- 使用文档指定的查询接口；

- 不使用高频无间隔轮询；

- 记录最后观察到的状态；

- 进入成功状态后执行结果层验证；

- 进入失败状态或观察超时后返回 failure；

- 文档没有定义状态接口或成功状态时返回 safe_blocked。

## 15. 重试规则

### 15.1 可以有限重试

- 文档或公共错误码明确标记为可重试的错误；

- 限流；

- 临时服务端错误；

- 临时网络错误；

- 幂等查询接口的客户端超时。

默认最多重试 3 次，采用退避等待，例如 1 秒、2 秒和 4 秒。API 文档另有要求时，以 API 文档为准。

### 15.2 不得自动重试

- 凭证或权限错误；

- 参数校验错误；

- 配额、库存或余额不足；

- 资源状态冲突；

- 需要用户选择的冲突；

- 非幂等写请求的结果不确定；

- 文档明确标记为不可重试的错误。

### 15.3 写请求超时

写请求超时后：

1. 不立即重复发送；

1. 使用原幂等令牌、资源名称、任务 ID 或查询接口确认是否已生效；

1. 只有文档保证幂等时才重试；

1. 无法确认时返回 safe_blocked，并保留现有证据。

## 16. 错误处理

错误处理优先使用：

1. 当前 API 的专有错误码；

1. 产品公共错误码；

1. HTTP 状态；

1. SDK 或 CLI 的结构化错误。

标准分类：

| 分类 | 默认行为 |
| --- | --- |
| authentication_error | 停止，按全局规范修复身份配置 |
| permission_denied | 停止，返回 safe_blocked |
| invalid_parameter | 修正确定性输入错误后新建调用尝试 |
| resource_not_found | 核对资源 ID 和地域，不得猜测 |
| resource_conflict | 停止并请求用户或上层任务决策 |
| quota_or_inventory_error | 停止，报告缺口 |
| throttled | 有限退避重试 |
| transient_service_error | 有限退避重试 |
| unknown_error | 停止，保存 RequestId 和脱敏错误 |

不得仅根据自然语言错误消息猜测根因。未知错误保持为未知。

## 17. 标准状态

| 状态 | 含义 |
| --- | --- |
| verified_success | API 请求、响应和结果层验证均通过 |
| safe_blocked | 缺少执行所需的输入、权限、文档或明确语义，未继续调用 |
| failure | 请求、响应、异步终态或结果验证失败 |

safe_blocked 和 failure 均不属于调用成功。

## 18. 标准输出

```json
{
  "status": "verified_success",
  "execution": {
    "requested_method": "auto",
    "actual_method": "sdk",
    "credential_source": "local_secure_environment",
    "endpoint_source": "official_product_documentation",
    "sdk_default_overridden": false,
    "switch_count": 0
  },
  "api": {
    "product": "<product>",
    "action": "<action>",
    "version": "<version>",
    "region": "<region-or-null>"
  },
  "request": {
    "method": "<method>",
    "endpoint": "<endpoint>",
    "parameters": {
      "<parameter>": "<sanitized-value>"
    }
  },
  "response": {
    "request_id": "<request-id>",
    "resource_id": "<resource-id-or-null>",
    "task_id": "<task-id-or-null>",
    "result": "<sanitized-result-or-summary>"
  },
  "verification": {
    "transport_passed": true,
    "api_passed": true,
    "result_passed": true,
    "final_state": "<state-or-null>"
  },
  "retry_count": 0,
  "warnings": []
}
```

失败输出至少包含：

```json
{
  "status": "failure",
  "api": {
    "product": "<product>",
    "action": "<action>",
    "version": "<version>"
  },
  "request_id": "<request-id-if-available>",
  "error": {
    "category": "<error-category>",
    "code": "<error-code>",
    "message": "<sanitized-message>",
    "retryable": false
  },
  "retry_count": 0,
  "next_action": "<required-action>"
}
```

输出不得包含凭证、签名、令牌、密码和未脱敏敏感数据。

## 19. 接口特有规则

如果某个 API 存在通用规范无法表达的特殊语义，应优先在原 API 文档中说明，例如：

- 特定参数组合决定不同结果字段；

- 空结果具有特殊业务含义；

- 创建接口返回 ID 后需要查询特定状态；

- 特定错误码允许重试；

- 部分字段仅在某种资源状态下有效。

API 页面可以在统一 Agent 入口后增加不超过三句的接口特有提示。示例：

```markdown
AI Agent：调用本接口前，请读取并遵守 [通用 API Agent 调用规范](api-agent-invocation.md)。
本接口为只读同步查询；返回空列表表示请求成功但没有匹配数据。
```

只有无法从原 API 文档可靠读取的特殊语义，才需要增加提示。不得为每个 API 再创建一篇重复的调用规范。

## 20. 完成检查表

- 已读取当前 API 文档，而不是仅依赖接口名称。

- 已提取 Action、Version、Service、方法、地址和地域。

- 已记录用户请求的执行方式和实际执行方式。

- 已验证实际 Endpoint 与官方文档推荐地址一致。

- 已检查全部必填参数和条件依赖。

- 未使用示例值、占位符或猜测值执行请求。

- 已按照全局规范检查身份、授权和风险条件。

- 已判断接口类型、同步性和幂等性。

- 已保存非空 RequestId，或说明接口未提供该字段。

- 已完成传输层、API 层和结果层验证。

- 异步接口已查询真实终态。

- 查询接口已正确处理空结果和分页。

- 重试没有违反幂等和错误分类要求。

- 输出已脱敏并使用标准状态。

适用检查项全部满足后，Agent 才能结束本次 API 调用。
