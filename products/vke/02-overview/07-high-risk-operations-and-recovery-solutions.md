在运行或部署容器服务相关业务过程中，用户可能会触发不同层面的高危操作，导致不同程度上的业务故障。为更好地帮助用户了解、预估和避免操作风险，本文将从集群/节点池/节点、网络与负载均衡、存储、日志、组件等多个维度出发，介绍相关高危操作、其影响以及恢复方案。

## 集群/节点池/节点

|分类 |高危操作 |影响 |恢复方案 |
|---|---|---|---|
|API Server |操作 API Server 关联的负载均衡实例（CLB）及其子资源（如监听器、后端服务器组）。 |集群相关资源不可操作。 |不可恢复。 |
|容器运行时 |自行使用/安装 Docker 等其他运行时到节点上。<br><br>> **说明**<br><br><br>> VKE 目前仅支持使用 containerd 运行时。<br> |可能导致节点不可用。 |先移除节点并选择保留 ECS 实例。再通过添加已有节点，将保留的 ECS 实例重新添加到 VKE 集群，完成节点初始化，恢复容器运行时到 VKE 支持的运行时。详细操作，请参见 [移除节点](https://www.volcengine.com/docs/6460/100953#.56e76Zmk6IqC54K5)、[添加已有节点](https://www.volcengine.com/docs/6460/100950)。 |
|Worker 节点 |修改集群内节点默认安全组。 |可能导致节点不可用。 |恢复默认安全组设置。节点默认安全组，请参见 [安全组设置](https://www.volcengine.com/docs/6460/97726)。 |
||自行修改节点核心组件（如 kubelet）参数。 |可能导致节点不可用。 |按照节点核心组件的默认配置进行恢复。 |
||节点被移除且未保留云服务器（ECS）实例。 |节点不可用。 |不可恢复。 |
||更改节点 IP。 |节点不可用。 |改回原来的节点 IP。 |
||重装操作系统。 |节点组件被删除。 |先移除节点并选择保留 ECS 实例。再通过添加已有节点，将保留的 ECS 实例重新添加到 VKE 集群。详细操作，请参见 [移除节点](https://www.volcengine.com/docs/6460/100953#.56e76Zmk6IqC54K5)、[添加已有节点](https://www.volcengine.com/docs/6460/100950)。 |
||修改操作系统配置。 |可能导致节点不可用。 |恢复原来的配置。 |
||更改节点对应 ECS 实例规格。 |可能导致节点 maxPods、系统资源预留等参数不符合预期。 |先移除节点并选择保留 ECS 实例。再通过添加已有节点，将保留的 ECS 实例重新添加到 VKE 集群。详细操作，请参见 [移除节点](https://www.volcengine.com/docs/6460/100953#.56e76Zmk6IqC54K5)、[添加已有节点](https://www.volcengine.com/docs/6460/100950)。 |
||修改节点内目录相关权限。 |可能导致节点不可用。 |恢复原来的权限配置。 |
||手动停止节点 kubelet 和 containerd 服务。 |节点不可用。 |重新启动节点的 kubelet 和 containerd 服务。 |
||升级/降级节点操作系统内核版本。 |可能导致节点异常或无法运行 Pod。 |恢复操作系统内核版本。 |
||在 ECS 实例系统盘中保存重要数据。 |移除节点时会导致系统盘数据丢失。 |无法恢复。 |
||通过 kubectl 命令移除节点。 |节点异常。 |移除节点后重新扩容。 |
||添加已有节点前未做数据备份。 |可能导致已有 ECS 实例上的数据丢失。 |提前做好数据备份。 |
||手动停止节点对应 ECS 实例。 |节点不可用。 |重新启动 ECS 实例。 |
|节点池 |移除节点池。 |节点池下所有节点都会被移除。 |无法恢复。 |
||创建节点池时选择自定义镜像，随后在 ECS 侧将自定义镜像删除。 |无法成功扩容出新节点。 |重新编辑节点池，选择默认可用镜像。 |
|其他 |修改或删除相关角色策略，如 AssumeRoleForVKE。 |可能导致集群功能不可用。 |重新设置角色策略。详细操作，请参见 [配置 IAM 用户权限](https://www.volcengine.com/docs/6460/101022)。 |

## 网络与负载均衡

> **说明**

> 网络与负载均衡相关内核参数，请根据您的具体操作系统，使用`sysctl -a`命令查看。

|高危操作 |影响 |恢复方案 |
|---|---|---|
|通过负载均衡控制台修改 VKE 管理的 CLB/ALB 配置，包括 CLB/ALB、监听及服务器组。 |导致 CLB/ALB 异常。 |恢复 CLB/ALB 的配置。 |
|修改内核参数：<br><br><br>* IPv4 集群：`net.ipv4.ip_forward=0`<br><br>* IPv6 集群：`net.ipv6.ip_forward=0` |网络不通。 |修改内核参数为：<br><br><br>* IPv4 集群：`net.ipv4.ip_forward=1`<br><br>* IPv6 集群：`net.ipv6.ip_forward=1` |
|修改内核参数：<br><br><br>* `net.ipv4.conf.all.rp_filter = 1\|2`<br><br>* `net.ipv4.conf.[eth*].rp_filter = 1\|2`<br><br><br>> **说明**<br><br><br>> `eth*`代表所有以`eth`开头的网卡。<br> |网络不通。 |修改内核参数为：<br><br><br>* `net.ipv4.conf.all.rp_filter = 0`<br><br>* `net.ipv4.conf.[eth*].rp_filter = 0` |
|修改内核参数：<br><br>`net.ipv4.vs.conn_reuse_mode = 0`<br><br>IPv6 集群修改内核参数：<br><br>`net.ipv6.conf.all.disable_ipv6=1` |在高并发、短连接流量，Service Pod 滚动升级的场景下，一定概率触发 Service 访问不通。<br><br>IPv6 不可用。 |修改内核参数为：<br><br>`net.ipv4.vs.conn_reuse_mode = 1`<br><br>修改内核参数为：<br><br>`net.ipv6.conf.all.disable_ipv6=0` |
|修改内核参数：<br><br><br>* `net.bridge.bridge-nf-call-ip6tables=0`<br><br>* `net.bridge.bridge-nf-call-iptables=0` |导致 NAT 网关异常。 |修改内核参数为：<br><br><br>* `net.bridge.bridge-nf-call-ip6tables=1`<br><br>* `net.bridge.bridge-nf-call-iptables=1` |
|修改内核参数<br><br>`net.ipv6.conf.[eth*].accept_ra=1` |学习不到 IPv6 路由，网络不通。 |修改内核参数：<br><br>`net.ipv6.conf.[eth*].accept_ra=2` |
|修改内核参数：<br><br>`net.ipv4.tcp_tw_recycle = 1` |导致 NAT 网关异常。 |修改内核参数为：<br><br>`net.ipv4.tcp_tw_recycle = 0` |
|移除 Service 中复用已有 CLB 的 Annotation，即`service.beta.kubernetes.io/volcengine-loadbalancer-id: ${YOUR_LB_ID}`。 |导致 CLB 异常。 |在 Service 中添加复用已有 CLB 的 Annotation。<br><br>> **说明**<br><br><br>> 复用已有 CLB 实例的 Service 无法直接修改为使用自动创建 CLB 的 Service。您需要重新创建 Service。<br> |

## 存储

|高危操作 |影响 |恢复方案 |
|---|---|---|
|在控制台手动解除挂载的云盘。 |Pod 写入报 I/O Error。 |重启 Pod，手动清理节点挂载残留。 |
|节点上 umount 磁盘挂载路径。 |Pod 写入本地磁盘。 |重启 Pod。 |
|节点上直接操作云盘。 |Pod 写入本地磁盘。 |不可恢复。 |
|手动删除 NAS 挂载目录。 |Pod 写入报 I/O Error。 |重启 Pod。 |
|删除正在使用的 NAS 盘或挂载点。 |Pod 出现 I/O Hang。 |重启 ECS 节点。 |

## 日志

|高危操作 |影响 |恢复方案 |
|---|---|---|
|删除宿主机`/var/lib/filebeat-registry`目录。 |日志重复采集。 |不可恢复。该目录下的文件记录了日志的采集位置。 |
|删除宿主机`/var/lib/filebeat-data`目录。 |日志丢失。 |不可恢复。该目录是待消费的日志缓存文件。 |
|删除 collectrule CRD 及其对应的资源。 |日志采集失效。 |重新创建被删除的 CRD 以及对应的资源，但失效期间的日志无法恢复。<br><br>> **说明**<br><br><br>> 删除 CRD 会关联删除对应所有的实例，即使恢复 CRD 后还需要手动创建被删除的实例。<br> |
|删除日志服务侧有 VKE Tag 的采集规则。 |日志采集失效。 |重建对应的 collectrule CRD 及其对应的资源。 |
|删除日志组件。 |日志采集失效。 |重新安装日志组件并手动恢复 collectrule CRD 及其对应的资源，但删除期间的日志无法恢复。<br><br>> **说明**<br><br><br>> 删除日志组件相当于删除 collectrule CRD 及其对应的资源和日志采集器 log\-collector，期间日志采集能力全部丢失。<br> |

## 组件

|高危操作 |影响 |恢复方案 |
|---|---|---|
|删除组件相关资源，如负载、配置项等。 |组件不可用。 |非系统必装组件可通过卸载再安装的方式修复，系统必装组件不可恢复。 |
|修改组件相关资源，如负载、配置项等。 |组件不可用或者相应配置被刷新。 |恢复组件原先配置，或者卸载再安装组件。如果组件为系统必装组件，则无法恢复。 |
|安装组件之前，在组件预安装命名空间中创建了和组件同名的资源。 |组件安装失败。 |删除目标命名空间中同名的资源，然后再安装组件。 |
