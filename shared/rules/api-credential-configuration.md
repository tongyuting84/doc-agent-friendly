---
spec_id: api-credential-configuration
spec_version: "2.0"
applies_to: [api_explorer, sdk, signed_http]
requires: [global-agent-execution]
---

# Agent API 凭证配置指南

> 适用范围：为 API Explorer、官方 SDK 和签名 HTTP 客户端准备火山引擎 API 身份凭证。API 操作必须同时遵守[全局 Agent 执行规范](global-agent-execution.md)和[API Agent 调用规范](api-agent-invocation.md)。

> 安全目标：让 Agent 能够调用 API，同时避免 AK、SK、令牌、Cookie 和签名进入聊天、代码、日志、截图或 Git 仓库。

## 1. 先选择执行方式

不同执行方式的认证环境相互独立：

| 执行方式 | 认证来源 |
| --- | --- |
| API Explorer | 当前浏览器登录会话 |
| 官方 SDK | 工作负载身份、Profile、临时凭证或安全环境变量 |
| 签名 HTTP | 访问密钥或临时安全凭证 |

浏览器已经登录不代表 SDK 或签名 HTTP 客户端已完成认证。不得从浏览器导出 Cookie、登录凭证、临时令牌或签名信息供其他客户端使用。

## 2. 凭证来源优先级

按以下顺序选择：

1. 工作负载身份、实例角色或平台托管身份；
2. 有明确有效期的临时安全凭证；
3. 受信任凭证存储或已配置的 Profile；
4. 仅在无法使用以上方式时，使用长期 AK/SK。

长期 AK/SK 应遵守最小权限、定期轮换和及时禁用要求。

## 3. 禁止事项

不得：

- 把 AK、SK、SessionToken、密码或 Cookie 发送到聊天；
- 把凭证写入源代码、Markdown、命令参数或 Git 仓库；
- 在日志、错误信息、截图或最终回复中显示凭证；
- 从 API Explorer 或控制台导出浏览器凭证供 SDK 或签名 HTTP 使用；
- 使用权限更高的身份绕过权限拒绝；
- 把文档示例值、账号 ID、用户名或变量名当作 Access Key ID。

## 4. 本地配置原则

- 优先使用操作系统凭证存储、官方 Profile 或安全环境注入；
- 只能检查凭证是否存在，不得打印其值；
- Access Key ID 和 Secret Access Key 必须来自同一组密钥；
- 交互式输入必须关闭回显；
- 如果命令包含交互式 read，不要把提示命令和后续命令整段粘贴，推荐使用封装脚本；
- 临时文件应位于非仓库目录，权限限制为仅当前用户可读写。

## 5. 临时凭证文件

只有在执行环境不能使用更安全的凭证链时才使用临时文件。

要求：

~~~yaml
temporary_credentials:
  location: non_repository_temporary_directory
  permissions: owner_read_write_only
  contents_logged: false
  deleted_after_execution: true
  deletion_verified: true
~~~

Agent 读取临时文件时不得回显内容。调用结束后，无论成功或失败，都必须删除文件并验证删除结果。删除失败时，应报告残留路径并要求用户人工处理。

## 6. 执行前检查

~~~yaml
credential_preflight:
  source_type: workload_identity | temporary_token | profile | secure_environment | temporary_file
  access_key_present: true
  secret_key_present: true
  session_token_present: true | false | not_applicable
  values_logged: false
  repository_path_used: false
  cleanup_required: true | false
~~~

格式检查只能发现空值、明显占位符和错误前缀，不能证明凭证真实有效。最终有效性以服务端认证结果为准。

## 7. 认证失败处理

认证失败时：

1. 停止请求，不自动重试；
2. 保存脱敏错误类别和 RequestId；
3. 核对使用的是 Access Key ID，而不是账号 ID、用户名或变量名；
4. 核对 AK 与 SK 是否属于同一组；
5. 核对临时凭证是否需要 SessionToken；
6. 核对调用方式是否使用了正确的凭证来源；
7. 不得通过导出浏览器凭证或切换到更高权限身份规避。

## 8. Agent 输出要求

Agent 只能输出凭证来源类型和检查状态：

~~~json
{
  "authentication": {
    "source": "local_secure_environment",
    "credentials_present": true,
    "credentials_logged": false,
    "temporary_file_cleanup": "complete"
  }
}
~~~

不得输出凭证值、长度以外的可识别片段、签名、Cookie 或令牌。
