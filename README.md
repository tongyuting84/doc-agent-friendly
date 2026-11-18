# Agent Friendly Documentation

本仓库集中管理面向 AI Agent 的产品技术文档、通用执行规则和文档模板。

目标是让 Agent 在获得必要上下文与合法授权后，仅根据仓库中的文档完成真实任务、验证结果，并在无法继续时安全停止。

## 内容结构

- [`shared/rules/`](shared/rules/)：跨产品复用的 Agent 执行规则。
- [`shared/templates/`](shared/templates/)：新增文档使用的模板。
- [`products/`](products/)：按产品及官网信息架构组织的 Markdown 文档。
- [`CATALOG.md`](CATALOG.md)：仓库级文档目录。

## 使用原则

1. 通用规则定义“如何安全执行”。
2. 产品文档定义“执行什么”和产品事实。
3. 业务文档保持原子化，原则上只引用全局规范；全局规范自动路由 API、控制台和 Kubernetes YAML 子规范。
4. 审核与实测报告不进入本线上文档仓库。
5. `main` 分支作为本仓库的唯一事实来源；原 Codebase 仓库保留迁移历史。

## 当前产品

- [Volcengine Kubernetes Engine（VKE）](products/vke/)

## Agent 入口

开始产品任务前先阅读[全局 Agent 执行规范](shared/rules/01-global-agent-execution.md)。Agent 必须按其中的路由表自动读取 API、凭证、控制台或 Kubernetes YAML 子规范。
