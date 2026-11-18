# Volcengine Kubernetes Engine

本目录按照 VKE 官网文档信息架构组织 Markdown 文档。

## 文档导航

1. [What's New](01-whats-new/)
2. [产品简介](02-overview/)
3. [Billing](03-billing/)
4. [Getting started](04-getting-started/)
5. [User guide](05-user-guide/)
6. [Best practices](06-best-practices/)
7. [Solution](07-solutions/)
8. [Security and compliance](08-security-and-compliance/)
9. [API reference](09-api-reference/)
10. [SDK reference](10-sdk-reference/)
11. [Development guide](11-development-guide/)
12. [FAQs](12-faqs/)

## Agent 执行入口

所有任务统一引用[全局 Agent 执行规范](../../shared/rules/01-global-agent-execution.md)。该规范会根据实际内容自动加载 API、凭证、控制台或 Kubernetes YAML 子规范。

产品文档负责 VKE 特有事实；通用执行、安全、凭证和验证规则不得复制到每篇文档中。
