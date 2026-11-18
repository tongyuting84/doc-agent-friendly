---
title: 产品功能
product: vke
source_url: https://docs.volcengine.com/docs/vke/features?lang=zh
source_last_updated: 2026-07-07T19:29:03+08:00
synced_at: 2026-11-18
---

# 产品功能

VKE 提供 Kubernetes 集群、节点、应用、网络、存储、可观测和开发者工具等管理能力。

## 权限

- IAM：控制火山引擎账号和 IAM 用户对 VKE 云资源的访问。
- RBAC：控制用户或角色对集群内 Kubernetes 资源的访问。

## 集群、节点池和节点

- 创建和管理托管集群、集群模板及注册集群。
- 管理 Kubernetes 版本升级和系统组件。
- 创建普通或托管节点池，添加已有 ECS 节点。
- 根据集群负载动态扩缩节点资源。

## 应用

- 支持 Deployment、StatefulSet、Job、CronJob、DaemonSet 等工作负载。
- 支持镜像、端口、环境变量、存储、健康检查和调度配置。
- 支持应用伸缩、滚动更新、灰度发布和蓝绿发布。
- 支持通过模板和 Helm Chart 部署应用。

## 存储

VKE 支持云盘、对象存储、文件存储和本地存储等类型，并通过 CSI、PV、PVC 和 StorageClass 对接 Kubernetes 存储体系。具体支持范围以存储文档及控制台为准。

## 网络

- 支持 Flannel 和 VPC-CNI 等容器网络模型。
- 支持 Kubernetes Service、Ingress 和 DNS。
- 可对接火山引擎负载均衡、NAT 网关和相关网络产品。

## 弹性、调度和异构计算

- 支持节点、工作负载、定时任务和事件驱动伸缩能力。
- 支持 GPU、mGPU 和 RDMA 等异构资源。
- 支持 Gang、Capacity、负载感知和拓扑感知等调度能力。

## 可观测、诊断和安全

- 支持日志采集、基础观测、全栈观测、应用性能观测和集群审计。
- 提供故障诊断、检查自愈、集群巡检和安全基线巡检。
- 功能可能依赖日志服务、托管 Prometheus、APMPlus 等其他云产品。

## 开发者服务

VKE 支持原生 YAML 与 `kubectl`、OpenAPI、Go/Java/Python SDK 和 Terraform。使用这些执行渠道时，应继续阅读对应参考文档和 Agent 子规范。

## 个人用户功能差异

个人用户可以使用 VKE，但 VCI、部分网络存储、APIG Ingress、高性能计算 GPU、托管节点池、部分套件及部分托管组件可能不可用。完整清单和最新支持状态请参见[官网产品功能](https://docs.volcengine.com/docs/vke/features?lang=zh)。
