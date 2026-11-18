---
spec_id: kubernetes-yaml-agent-execution
spec_version: "1.0"
applies_to: [kubernetes_yaml, kubectl]
requires: [global-agent-execution]
---

# Kubernetes YAML Agent 执行规范

> 适用于 Agent 识别文档中的 Kubernetes YAML，检查本地与集群环境，使用 `kubectl` 校验、执行并验收资源。通用输入、风险、确认和证据规则由[全局 Agent 执行规范](01-global-agent-execution.md)定义。

## 1. 触发条件与边界

文档出现 Kubernetes 资源清单、`apiVersion`、`kind`、`kubectl apply/create/delete/patch` 等内容时必须加载本规范。

本规范当前只覆盖 Kubernetes YAML 和 `kubectl`，不泛化到云厂商 CLI、Helm、Terraform 或其他命令行工具。出现未覆盖工具时，应读取其独立规范；不存在规范时返回 `safe_blocked`。

## 2. 标准流程

```text
提取 YAML 和期望结果
→ 检查 kubectl
→ 定位 kubeconfig
→ 枚举并确认 Context、Cluster 和 Namespace
→ 检查集群连通性与权限
→ 校验 YAML、依赖和风险
→ 执行 client/server dry-run 与 diff
→ 展示变更并获得必要确认
→ 执行 YAML
→ 等待资源就绪
→ 验收实际状态并输出台账
```

## 3. 本地工具检查

执行前检查 `kubectl` 是否存在并读取客户端版本，例如：

```bash
command -v kubectl
kubectl version --client
```

未安装时不得假设路径或直接执行 YAML。应根据当前操作系统和可信官方文档提供安装方案；未经用户授权，不得自动安装需要管理员权限或修改系统环境的工具。

## 4. kubeconfig 与目标集群

按以下顺序定位配置：

1. 用户明确指定的 kubeconfig；
2. `KUBECONFIG` 环境变量中的路径列表；
3. 默认 `$HOME/.kube/config`。

只能检查文件是否存在、权限是否合理及其非敏感元数据，不得输出证书、客户端密钥或令牌。

使用只读命令识别：

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config view --minify
```

执行前必须明确并展示：

- kubeconfig 来源；
- 当前或拟使用的 Context；
- Context 指向的 Cluster；
- API Server 地址；
- 默认或目标 Namespace；
- 与文档要求的地域、集群和环境是否一致。

不得因为存在当前 Context 就假设它是目标集群。多个候选、目标不一致或无法识别时必须让用户选择；不得静默切换 Context。优先通过每条命令的 `--context` 和 `--namespace` 显式限定目标，避免修改用户全局当前 Context。

## 5. 连通性与权限预检

使用只读方式验证集群连接，例如：

```bash
kubectl --context <context> cluster-info
kubectl --context <context> get namespaces
```

然后根据 YAML 的资源类型、Namespace 和预期动作用 `kubectl auth can-i` 动态检查权限。不得只检查一个固定资源，也不得因为能够读取 Namespace 就推断具备写权限。

认证失败、集群不可达、证书错误、权限不足或目标集群不一致时返回 `safe_blocked`，不得通过切换高权限身份规避。

## 6. YAML 静态检查

逐个对象检查：

- YAML 语法、多文档分隔和字段结构；
- `apiVersion`、`kind`、`metadata.name` 和 Namespace；
- 占位符、示例值、变量和未解析模板；
- 镜像、端口、资源请求与限制、标签和选择器；
- Secret、ConfigMap、ServiceAccount、PVC、CRD 等依赖；
- 已弃用或目标集群不支持的 API；
- ClusterRole、ClusterRoleBinding、特权容器、hostPath、hostNetwork、NodePort、LoadBalancer 等高风险配置；
- 对现有资源的覆盖、删除、不可变字段和所有权冲突。

Secret 的明文或 base64 内容均视为敏感信息，不得回显或写入报告。发现真实秘密被嵌入待提交文件时，应停止并要求改用受控方式。

## 7. Dry-run、Diff 与确认

先做本地校验：

```bash
kubectl apply --dry-run=client -f <file>
```

集群连通且权限允许时，再做服务端校验和差异检查：

```bash
kubectl --context <context> --namespace <namespace> apply --dry-run=server -f <file>
kubectl --context <context> --namespace <namespace> diff -f <file>
```

`dry-run=server` 会访问集群但不持久化资源。`kubectl diff` 返回码 `1` 通常表示存在差异，不应直接判为命令失败；必须结合输出语义判断。

执行前确认单至少包含 Context、Cluster、API Server、Namespace、文件或对象来源、将创建/修改/删除的对象、权限/公网/存储/费用影响、校验结果和验收方法。

## 8. 执行规则

- 默认使用文档明确指定的命令；文档以声明式 YAML 为主时优先 `kubectl apply`。
- 命令显式带 `--context` 和 `--namespace`；集群级对象标记 Namespace 为不适用。
- 不将 YAML 通过未经审查的 shell 插值直接拼接执行。
- 不用 `--validate=false`、`--force`、`--grace-period=0` 等选项绕过失败，除非文档明确要求、风险已解释且用户批准。
- 超时或连接中断后先查询对象和事件，不直接重复 apply/create。
- 删除、替换、强制接管字段所有权和修改集群级权限属于高风险操作，必须单独确认。

命令退出码为 0 只代表客户端命令完成，不代表工作负载可用。

## 9. 动态验收

根据资源类型选择查询与等待方式：

- Deployment/DaemonSet/StatefulSet：检查 rollout、期望副本、Ready/Available 和 Pod 状态；
- Job：等待 Complete 或明确 Failed；
- Pod：检查 Running、Ready、重启次数和事件；
- Service/Ingress：检查地址、端口、后端和文档要求的可达性；
- PVC：检查 Bound 及相关事件；
- CRD/自定义资源：使用文档或 CRD 定义的 Conditions 和状态字段。

必须检查 `ImagePullBackOff`、`CrashLoopBackOff`、调度失败、探针失败、权限拒绝和依赖缺失等异常。仅存在对象不等于验收通过。

## 10. 输出与清理

结果至少包含：

```yaml
status: verified_success | verified_success_with_warnings | safe_blocked | failure
kubernetes:
  context: <context>
  cluster: <cluster>
  api_server: <sanitized-server>
  namespace: <namespace-or-not-applicable>
  kubectl_version: <client-version>
validation:
  client_dry_run: passed
  server_dry_run: passed | not_applicable
  diff_reviewed: true
resources:
  created: []
  updated: []
  unchanged: []
verification: []
cleanup_status: not_requested
```

只清理由本次操作明确创建且得到授权的对象。清理前再次核对 Context、Namespace、精确对象和依赖；不得用宽泛标签或模糊名称删除用户原有资源。
