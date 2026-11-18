本文主要介绍使用容器服务过程中存在的一些限制条件。

## 限制概述

使用容器服务产品前，需要注意以下使用限制：

* 创建容器服务集群前，需要确保已经完成 [实名认证](https://console.volcengine.com/user/authentication/detail/)。

* 只有当您的账户余额和代金券可用额度总值大于等于 100 元，且除去代金券后账户余额不为零时，才能创建按量计费的云资源。

* 容器服务集群创建完成后，暂不支持以下操作：

   * 更改集群所属的私有网络 VPC。

   * 更改集群的容器网络 CNI 插件。

   * 更改集群内的容器网络 CIDR。

   * 卸载集群默认的系统必装组件。

   * 在不同命名空间下迁移应用。

* 容器服务集群所涉及的相关云基础资源的限制如下：

   * 在容器服务中仅支持创建和添加计费类型为 **按量计费** 的云资源（节点池除外），暂不支持计费类型为 **包年包月** 的云资源；使用已有云资源时，无计费类型限制。

   * 由于云服务器 ECS 等底层云资源的配额及库存限制，创建、扩容容器集群时，可能会出现只有部分节点创建成功的情况。

   * 由于公网 IP 受线路类型的配额及容量限制，创建公网类型的负载均衡器、NAT 网关资源时，可能会出现关联资源创建失败的情况。

   * 删除容器服务集群时，默认会同步删除所关联使用的按量计费的云资源，由于云资源的使用主体和权限可能不仅限于容器集群本身，会存在容器集群删除后，部分云资源残留的情况，可自行通过云资源控制台去删除残留资源。

## 集群配额限制

开通容器服务后，每个火山引擎账号（含其下子用户）默认的集群配额限制如下表所示。当前账号下详细的集群资源配额查询方法，请参见本文下方 [配额查看和申请](https://www.volcengine.com/docs/6460/79642?TagIDs=2%2C255#SAoYyaXa)。

> **注意**

> 使用容器服务过程中，若配额达到上限，您需要提升配额时，请务必确认：集群内因配额不足导致 Pending 状态的 Pod 数量。

> 若 Pending 状态的 Pod 数量过多，建议删除该状态的 Pod 再提升配额（或逐步少量额度提升配额），以免在提升配额后大批量 Pending 状态的 Pod 同时并发，导致服务瞬间压力过大，影响业务可用性。

|**限制项** |**单账号限制** |**例外申请方式** |
|---|---|---|
|实名认证 |企业或个人实名认证 |无 |
|创建按量计费资源的限制 |账户余额和代金券可用额度的总值大于等于 100 元 |无 |
|单地域下可创建的集群总数 |* 华北 2（北京）：4 个<br><br>* 华南 1（广州）：2 个<br><br>* 华东 2（上海）：2 个<br><br>* 亚太东南（柔佛）：2 个<br><br>* 亚太东南（雅加达）：2 个<br><br>* 中国香港：2 个 |[配额中心提交申请](https://www.volcengine.com/docs/6460/79642?TagIDs=2%2C255#SAoYyaXa) |
|单集群下可创建的节点总数 |* 华北 2 (北京)：100 个<br><br>* 华南 1 (广州)：100 个<br><br>* 华东 2 (上海)：100 个<br><br>* 亚太东南（柔佛）：100 个<br><br>* 亚太东南（雅加达）：100 个<br><br>* 中国香港：100 个 |[配额中心提交申请](https://www.volcengine.com/docs/6460/79642?TagIDs=2%2C255#SAoYyaXa) |
|单集群下可创建的节点池总数 |* 华北 2 (北京)：10 个<br><br>* 华南 1 (广州)：10 个<br><br>* 华东 2 (上海)：10 个<br><br>* 亚太东南（柔佛）：10 个<br><br>* 亚太东南（雅加达）：10 个<br><br>* 中国香港：10 个 |[配额中心提交申请](https://www.volcengine.com/docs/6460/79642?TagIDs=2%2C255#SAoYyaXa) |
|单集群可生成的 KubeConfig 证书总数 |火山引擎账号（主账号）及其下子用户分别：<br><br><br>* 私网访问 KubeConfig：5 个。<br><br>* 公网访问 KubeConfig：5 个。<br><br><br>> **说明**<br><br><br>> 主账号和子用户配额分开，相互不影响。<br> |无 |
|单集群可添加的资源标签（Tag）总数 |50 个 |无 |

## Kubernetes 集群规模

容器服务目前支持的 Kubernetes 集群（单账号单集群）规模如下表所示。

> **说明**

> 大规模集群相关更多资源大小和数量限制，请参见 [大规模集群限制和建议](https://www.volcengine.com/docs/6460/2389135)。

|分类 |默认规模 |最大支持规模 |最大规模申请方式 |
|---|---|---|---|
|节点数量 |* VPC\-CNI 模型集群：100 个<br><br>* Flannel 模型集群：200 个 |* VPC\-CNI 模型集群：5000 个<br><br>* Flannel 模型集群：1000 个 |联系您火山引擎容器服务业务相关对接人员。 |
|Pod 数量 |3000 个 |* 启用 NetworkPolicy：50000 个<br><br>* 不启用 NetworkPolicy：150000 个 ||
|ConfigMap/Secret 数量 |600 个 |12000 个 ||
|PV/PVC 数量 |2000 个 |60000 个 ||
|Service 数量 |200 个 |6000 个 ||
|Custom Resource 数量 |2000 个 |60000 个 ||
|ETCD 容量 |8 GiB |16 GiB ||
|ConfigMap/Secret/Custom Resource 单个资源大小 |\- |1 MiB ||
|每种类型 ETCD 对象的总大小 |200 MiB |800 MiB ||

## 节点池配额限制

单个节点池配额限制如下表所示。当前账号下详细的节点池资源配额查询方法，请参见本文下方 [配额查看和申请](https://www.volcengine.com/docs/6460/79642?TagIDs=2%2C255#SAoYyaXa)。

|**限制项** |**单账号限制** |**例外申请方式** |
|---|---|---|
|单节点池可添加的资源标签（Tag）总数 |50 个 |无 |
|单节点池可添加的 ECS 实例标签（ECS Tag）总数 |20 个 |无 |
|单节点池可关联的子网总数 |8 个 |无 |
|单节点池可关联的安全组总数 |5 个（含默认安全组） |无 |

## 依赖底层云产品配额限制

|**限制类** |**限制项** |**单账号限制** |**例外申请方式** |
|---|---|---|---|
|计算 |单次创建云服务器（ECS）的最大数量 |100 台 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单台云服务器实例可挂载的网卡数量（包括 1 张主网卡） |不同规格云服务器实例可挂载网卡的数量不同，具体请参见 [规格实例清单](https://www.volcengine.com/docs/6396/68527) |无 |
|网络 |单个账号单个地域可以创建的私有网络（VPC）数量 |10 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个私有网络下可创建的子网数量 |100 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个私有网络支持创建的路由表数量 |10 个（包括 1 个系统路由表，9 个自定义路由表） |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个路由表支持创建的自定义路由条目数量 |200 条 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个账号单个地域可创建的弹性网卡（ENI）数量 |100 张 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个账号单个地域下可创建的公网 IP（EIP）数量 |20 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个账号单个地域下可创建的安全组数量 |300 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||单个安全组中支持添加的规则数量（包括入方向规则和出方向规则） |200 条 |无 |
|块存储 |单个账号单个地域下可使用的单个云盘类型最大使用量 |64 TiB |[提交工单申请](https://console.volcengine.com/workorder/create) |
|负载均衡 |可创建的 CLB 实例数量 |60 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||每个 CLB 实例可关联的后端服务器总次数 |200 次<br><br>> **说明**<br><br><br>> 在不同后端服务器组中反复添加同一后端云服务器（IP相同则视为同一后端云服务器）会重复计算次数。<br> |[提交工单申请](https://console.volcengine.com/workorder/create) |
||每个 CLB 实例可添加的监听器数量 |50 个 |[提交工单申请](https://console.volcengine.com/workorder/create) |
||每个 HTTP 或 HTTPS 监听器可添加的转发规则数量 |50 条<br><br>> **说明**<br><br><br>> 默认转发规则不计入在内。<br> |[提交工单申请](https://console.volcengine.com/workorder/create) |
||每个监听器可关联的访问控制策略组数量 |5 个 |无 |

## 配额查看和申请

容器服务对接了火山引擎配额中心，支持查看账号下的容器服务各资源用量，同时部分资源也支持提交扩充配额的申请。

1. 登录 [配额中心](https://console.volcengine.com/quota/productList)。

2. 在 **产品列表** 页面搜索 **容器服务**，并单击 **容器服务**。

   ![图片](https://portal.volccdn.com/obj/volcfe/cloud-universal-doc/upload_3fb39e69fee10ce1f74315b235524d3f.png)

3. 在容器服务配额页面，查看弹性容器实例相关的配额参数。

   您可以根据实际情况，对支持扩充配额的资源提交扩充配额申请。

   ![图片](https://portal.volccdn.com/obj/volcfe/cloud-universal-doc/upload_ba51d4cc5b52c4b10ad33285b06e9737.png)
