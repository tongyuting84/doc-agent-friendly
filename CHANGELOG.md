# Changelog

## 2026-11-17

- 将通用规则重构为“全局统一入口 + 操作类型子规范”。
- 全局规范新增 API、控制台和 Kubernetes YAML 的强制路由规则。
- API 规范移除 CLI 渠道，并明确与凭证指南的依赖关系。
- 将控制台规则重命名并去除产品专属示例。
- 新增 Kubernetes YAML Agent 执行规范，覆盖 kubectl、kubeconfig、Context、集群连通性、权限、dry-run、diff 和动态验收。
- 产品入口和模板改为只引用全局规范。

## 2026-11-16

- 初始化 Agent Friendly 多产品文档仓库。
- 按 VKE 官网信息架构建立产品目录。
- 从 `code.byted.org/tongyuting/doc-use` 迁移四份通用规则，并将规则间依赖改为仓库内相对链接。
- 增加仓库目录、贡献指南和文档模板。
