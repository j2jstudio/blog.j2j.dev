---
title: "了解Prometheus"
date: 2020-09-23T09:25:26+08:00
draft: true
---

## Prometheus 

官网的介绍


Prometheus 是一个开源系统监控和警报工具包，最初是在SoundCloud构建的。自2012年成立以来，许多公司和组织都采用了 Prometheus，该项目拥有非常活跃的开发人员和用户社区。它现在是一个独立的开源项目，独立于任何公司进行维护。为了强调这一点，并澄清项目的治理结构，Prometheus 在2016年加入了 Cloud Native Computing Foundation，这是继Kubernetes之后的第二个托管项目。


<!--more-->

主要的组件：

*  Prometheus server ，它收集和存储时间序列数据
* 用于检测应用程序代码的客户端库（client libraries）
* 支持短期工作的推送网关（push gateway ）
* 提供服务的特定目的的 导出接口（ exporters），如HAProxy、StatsD、Graphite等。
* 处理警报的 alertmanager
* 各种支持工具

架构：

![Prometheus Architecture](https://prometheus.io/assets/architecture.png)


## 数据获取、存储

1. 通过 instrumented jobs 抓取 metrics 数据 （pull 模式）
2. 通过推送网关 push gateway 接收数据。（push 模式)

数据获取后，会全部存储到 Prometheus 本地数据库中。Prometheus 记录的为时间序（time series）数据。 它既适用于以机器为中心的监视，也适用于对高度动态的面向服务的体系结构的监视。 同时 Prometheus 收集的数据不够详细和完整，不会记录每一个请求信息（比如per-request billing），如果需要100%的准确度需要事情其他系统。

Prometheus 抓取的时间间隔默认为15s。