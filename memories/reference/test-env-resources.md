---
name: 银享惠测试环境资源清单
description: 测试环境所有基础资源（Nacos/Redis/MySQL/RabbitMQ/K8s/ELK等）的地址、端口、凭证，来自副本-银享惠测试环境资源需求列表.xlsx
type: reference
---

# 银享惠测试环境资源清单

> **规则**：仅用于只读查询，任何修改操作必须先征求用户同意。
> 来源：`E:\work\副本-银享惠测试环境资源需求列表.xlsx`

---

## 核心中间件

| 服务 | 内网地址 | 管理地址 | 凭证 |
|------|----------|----------|------|
| **Nacos** | `10.213.60.22/32/33` | `http://10.213.91.12:8848/nacos` | 通过 Nacos API 查询 |
| **RabbitMQ** | `10.213.91.11:5672`（负载）<br>节点: `10.213.60.8/9/10` | `http://10.213.60.8:15672/` | `admin / In7#ynirb`<br>vhost=`/prd_k8s`, user=`k8s`, pwd=`5rfvbgt6NHY&` |
| **Redis** | `10.213.60.34/29/31` | — | 集群模式 5.0，代理模式 4分片4G |
| **K8s (Rancher)** | master: `10.213.60.54/55/56`<br>worker: `10.213.60.2/11/12` | `https://rancher.yinxiangpay.ysidc` | `Shell-only / Q8+4Qo7gzk=0NWSt`<br>或 `yxh-fat / 123456` |
| **Kafka** | `10.213.60.23:9092, 10.213.60.25:9092, 10.213.60.27:9092` | — | 版本 2.13-3.3.1 |
| **ELK** | es: `10.213.60.24/30/28:9200`<br>kibana: `10.213.60.5:5601`<br>logstash: `10.213.60.7` | `http://10.213.60.5:5601/` | es: `elastic / 5rfvbgt6NHY&` |

## 数据库

| 实例 | 地址 | 数据库/账号 |
|------|------|------------|
| **业务MySQL** | `10.213.60.18:3306` | `posx_tcs` → `ys_tcs / VkZO2cuVKqBafAay`<br>`posx_prd`(交易) → `ys_trans / RXQk7dz8eiPuWeBJ`<br>`posx_bss` → `ys_bss / Iw2bihjFbJ8lWKFX` |
| **CAT/XXL-JOB MySQL** | `10.213.60.13:3306` | `cat` → `ys_cat / hxOPx9QXMF3mc8Rb`<br>`xxl_job` → `ys_xxl / YyAmKcpAJR8oNBv4` |

## 定时任务 & 监控

| 服务 | 地址 | 凭证 |
|------|------|------|
| **XXL-JOB** | `http://10.213.60.15:11004/posx-job-admin`<br>公网: `xxljob.yinxiangpay-test.com:11004` | `admin / Ys#20251009` |
| **CAT** | `http://10.213.60.21:8080/cat` | `admin / admin` |

## CI/CD & 仓库

| 服务 | 地址 | 凭证 |
|------|------|------|
| **Jenkins** | `http://10.213.60.16:8080` | `admin / 5rfvbgt6NHY&` |
| **GitLab**(测试) | `http://10.213.60.19` | — |
| **Nexus**(测试) | `http://10.213.60.17:8081` | — |
| **Harbor** | `http://10.213.60.26` | `admin / 5rfvbgt6NHY&` |

## 存储 & 文件

| 服务 | 地址 | 凭证 |
|------|------|------|
| **SFTP** | `10.213.60.37:22`<br>公网: `58.250.168.179:22022` | `yxh-sftp / 0okBcsULj5w0`<br>`jxyh-sftp / aJ6%bA4$r` |
| **OSS** | `test-yinxinghui-public.oss-cn-shenzhen.aliyuncs.com`<br>`test-yinxinghui-private.oss-cn-shenzhen.aliyuncs.com` | — |
| **NAS** | `10.213.60.20:/data` | — |
| **WebDAV** | `webdav.yinxiangpay.ysidc:8848` | — |

## 业务系统

| 系统 | 测试地址 | 测试账号 |
|------|----------|----------|
| **OMS综合管理** | `https://oms.yinxiangpay-test.com/` | `ceshi003 / ceshi003*` 或 `admin / admin@123` |
| **惠享拓(代理)** | `https://agent.yinxiangpay-test.com/` | `15855558888 / a123456789.` |
| **SFTP前端Nginx** | `http://10.213.60.14/` | — |
| **ShardingProxy** | `10.213.60.35` | — |
