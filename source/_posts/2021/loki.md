---
title: loki
layout: post
categories: ops
tags: ops
abbrlink: 21442
date: 2021-09-22 10:43:31
---

# 简 介

Loki是受Prometheus启发由Grafana Labs团队开源的水平可扩展，高度可用的多租户日志聚合系统。 开发语言: Google Go。它的设计具有很高的成本效益，并且易于操作。使用标签来作为索引，而不是对全文进行检索，也就是说，你通过这些标签既可以查询日志的内容也可以查询到监控的数据签，极大地降低了日志索引的存储。系统架构十分简单，由以下3个部分组成 ：

* Loki 是主服务器，负责存储日志和处理查询 。
* promtail 是代理，负责收集日志并将其发送给 loki（只收集日志，不在本地进行运算） 。
* Grafana 用于 UI 展示。
只要在被监控服务器上安装promtail来收集日志然后发送给Loki存储，就可以在Grafana UI界面通过添加Loki为数据源进行日志查询（如果Loki服务器性能不够，可以部署多个Loki进行存储及查询）。作为一个日志系统不关只有查询分析日志的能力，还能对日志进行监控和报警


<!--more-->

## 前言

   为什么会考虑到loki呢，我们公司使用的是阿里云的SLS，每个月的费用大概是11W左右，开销太大了，自己搭建ELK，组件又多，但是我们只是使用了里面的查询功能，有点大材小用了，我们的需求只是要查日志，报警足以，所以就搭建了loki日志查询

## 体会

   经过一段时间的使用，loki确实要比ELK轻很多，查询也比之前快了很多（没有超多1T数据测试，后续测试），而且费用也减少了很多

### 优点

    低索引开销
    * loki 和 es 最大的不同是 loki只对标签进行索引而不对内容索引
    * 这样做可以大幅降低索引资源开销 (es 无论你查不查，巨大的索引开销必须时刻承担)

## 本地化安装

### 集群情况

|      IP      | hostname |    部署软件    |
| :----------: | :------: | :------------: |
| 10.181.0.112 |  loki01  | loki、promtail |
| 10.181.0.96  |  loki02  | loki、promtail |
| 10.181.0.28  |  loki03  | loki、promtail |

#### 1、下载安装包

``` 
# 下载promtail和loki二进制包
wget  https://github.com/grafana/loki/releases/download/v2.2.1/loki-linux-amd64.zip
wget https://github.com/grafana/loki/releases/download/v2.2.1/promtail-linux-amd64.zip  
```

#### 2、安装 promtail

```
mkdir /opt/promtail -pv
# promtail配置文件
cat <<EOF> /opt/promtail/promtail.yaml  
server:  
  http_listen_port: 9080  
  grpc_listen_port: 0  
  
positions:  
  filename: /var/log/positions.yaml
client:  
  url: http://localhost:3100/loki/api/v1/push  
  
scrape_configs:  
 - job_name: system  
   pipeline_stages:  
   static_configs:  
   - targets:  
      - localhost  
     labels:  
      job: varlogs    
      host: yourhost # A `host` label will help identify logs from this machine vs others  
      __path__: /var/log/*.log  # The path matching uses a third party library: https://github.com/bmatcuk/doublestar  
EOF

# 增加service文件  
cat <<EOF >/etc/systemd/system/promtail.service  
[Unit]  
Description=promtail server  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
ExecStart=/opt/promtail/promtail -config.file=/opt/promtail/promtail.yaml  
StandardOutput=syslog  
StandardError=syslog  
SyslogIdentifier=promtail  
[Install]  
WantedBy=default.target  
EOF  
systemctl daemon-reload  
systemctl restart promtail  
systemctl status promtail
```

#### 安装loki

```
mkdir /opt/loki -pv  
  
# promtail配置文件  
$ cat <<EOF> /opt/loki/loki.yaml  
auth_enabled: false  
  
server:  
  http_listen_port: 3100  
  grpc_listen_port: 9096  
  
ingester:  
  wal:  
    enabled: false
    dir: /opt/loki/wal  
  lifecycler:  
    address: 127.0.0.1  
    ring:  
      kvstore:  
        store: inmemory  
      replication_factor: 1  
    final_sleep: 0s  
  chunk_idle_period: 1h       # Any chunk not receiving new logs in this time will be flushed  
  max_chunk_age: 1h           # All chunks will be flushed when they hit this age, default is 1h  
  chunk_target_size: 1048576  # Loki will attempt to build chunks up to 1.5MB, flushing first if chunk_idle_period or max_chunk_age is reached first  
  chunk_retain_period: 30s    # Must be greater than index read cache TTL if using an index cache (Default index read cache TTL is 5m)  
  max_transfer_retries: 0     # Chunk transfers disabled  
  
schema_config:  
  configs:  
    - from: 2021-09-19
      store: boltdb-shipper  
      object_store: filesystem  
      schema: v11  
      index:  
        prefix: index_  
        period: 24h  
  
storage_config:  
  boltdb_shipper:  
    active_index_directory: /opt/loki/boltdb-shipper-active  
    cache_location: /opt/loki/boltdb-shipper-cache  
    cache_ttl: 24h         # Can be increased for faster performance over longer query periods, uses more disk space  
    shared_store: filesystem  
  filesystem:  
    directory: /opt/loki/chunks  
  
compactor:  
  working_directory: /opt/loki/boltdb-shipper-compactor  
  shared_store: filesystem  
  
limits_config:  
  reject_old_samples: true  
  reject_old_samples_max_age: 168h  
  
chunk_store_config:  
  max_look_back_period: 0s  
  
table_manager:  
  retention_deletes_enabled: false  
  retention_period: 0s  
  
ruler:  
  storage:  
    type: local  
    local:  
      directory: /opt/loki/rules  
  rule_path: /opt/loki/rules-temp  
  alertmanager_url: http://localhost:9093  
  ring:  
    kvstore:  
      store: inmemory  
  enable_api: true  
EOF  
  
# service文件  
  
cat <<EOF >/etc/systemd/system/loki.service  
[Unit]  
Description=loki server  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
ExecStart=/opt/loki/loki -config.file=/opt/loki/loki.yaml  
StandardOutput=syslog  
StandardError=syslog  
SyslogIdentifier=loki  
[Install]  
WantedBy=default.target  
EOF  
  
$ systemctl daemon-reload  
$ systemctl restart loki  
$ systemctl status loki
```

