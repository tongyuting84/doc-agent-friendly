---
title: 开服地域
product: vke
source_url: https://docs.volcengine.com/docs/vke/regions-and-availability-zones?lang=zh
synced_at: 2026-11-18
dynamic_content: true
---

# 开服地域

地域（Region）是物理数据中心所在的地理区域。资源创建后通常不能更换地域，不同地域之间的内网相互隔离。可用区（Availability Zone）是同一地域内相互隔离的物理区域，同一地域的可用区通过低时延内网连接。

## 选择建议

- 优先选择靠近用户或业务的地域，以降低访问时延。
- 对容灾要求较高的业务，可将资源部署到同一地域的不同可用区。
- 对节点间时延敏感的业务，可优先使用同一可用区。
- 不同地域的价格、库存、配额和功能开放情况可能不同。

## 当前地域

| 地域 | RegionID | 已公开可用区 |
|---|---|---|
| 华北 2（北京） | `cn-beijing` | `cn-beijing-a`、`cn-beijing-b`、`cn-beijing-c`、`cn-beijing-d` |
| 华南 1（广州） | `cn-guangzhou` | `cn-guangzhou-a`、`cn-guangzhou-b`、`cn-guangzhou-c` |
| 华东 2（上海） | `cn-shanghai` | `cn-shanghai-a`、`cn-shanghai-b`、`cn-shanghai-c`、`cn-shanghai-d`、`cn-shanghai-e` |
| 中国香港 | `cn-hongkong` | `cn-hongkong-a`、`cn-hongkong-b` |
| 亚太东南（柔佛） | `ap-southeast-1` | `ap-southeast-1a`、`ap-southeast-1b`、`ap-southeast-1c` |
| 亚太东南（雅加达） | `ap-southeast-3` | `ap-southeast-3a`、`ap-southeast-3b` |

> 地域和可用区属于动态信息。执行任务前必须通过官网、控制台或已授权的只读接口核对当前开放情况，不得只依赖本页快照。
