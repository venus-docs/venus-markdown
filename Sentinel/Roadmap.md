## Overview

Roadmap 时间点：

| 版本 | Milestone | 主要特性 |
| -------- | -------- | -------- |
| ✅0.2.0     | **2018.9** | 异步调用支持、热点参数限流、黑白名单功能 |
| 0.4.0     | **2018.11** | 集群限流（基础版） |
| 0.6.0     | **2018.12** | 指标/监控 API 标准化，适配 Prothemeus 等 |
| 0.8.0     | **2019.1** | Reactive, Functional 支持 |
| 1.0.0     | **2019.2** | Service Mesh 支持 |

详细 Roadmap:

| 功能点 | 优先级 | 重要性 | Milestone 版本 |
| -------- | -------- | -------- | -------- |
| 注解支持     | 高     | 高     | 0.1.1 |
| ⚡️异步调用支持     | 高     | 高     | 0.2.0 |
| ⚡️热点参数限流   | 高     | 中     | 0.2.0 |
| ⚡️集群限流（基础版）  | 高   | 高  | 0.4.0 |
| 黑白名单功能     | 中     | 中     | 0.4.0 |
| 与更多主流框架适配   | 中     | 高     | 流动进行 |
| 与更多的数据源适配   | 中     | 高     | 流动进行 |
| ⚡️指标/监控 API 标准化  | 高     | 高     | 0.6.0（长期） |
| ⚡️Reactive 支持   | 中     | 高     | 0.8.0（长期） |
| 高版本 Java 适配（JDK 9+）     | 低     | 中   | 流动进行 |
| Service Mesh 支持  | 低   | 高  | 1.0.0（长期） |
| 多语言支持（集群限流客户端） | 低     | 中  | 长期 |
| 智能化（动态规则调配） | 低     | 中  | 1.1.0（长期） |

开源生态：

![Sentinel 未来的开源生态](https://user-images.githubusercontent.com/9434884/46240214-9c72ff80-c3d6-11e8-937a-0cffa1e8dc58.png)

## 异步调用支持

- Milestone: 0.2.0
- Motivation：未来各种 RPC 框架、Web 框架都朝着异步化的目标发展（Dubbo 3.0, gRPC, Spring WebFlux, Vert.x, 异步 Servlet, Netty 服务等），整个 Java 的发展方向也在朝着异步、响应式演进（无论是 CompletableFuture, Reactive Streams 还是后面的 Project Loom 协程），因此支持异步调用非常重要。
- 目标：支持异步调用链路的指标统计

## 热点参数限流

- Milestone: 0.2.0
- Motivation：将内部的热点参数限流功能加入到开源版本，同时进行优化，可以解决热点访问的问题。

## 集群限流与 Service Mesh

- Milestone: 0.4.0
- Motivation：引入集群限流的功能，与单机限流相组合形成两道屏障。同时，集群限流可以作为服务对外提供，可以为 Service Mesh 提供集群流量防护的服务。

## 监控标准化

- Milestone: 0.6.0
- Motivation：目前 Sentinel 的监控统计指标提供方式还可以更丰富（比如 percentile, gauge, histogram 等 metrics 信息）。同时监控 API 比较简单，缺乏对外提供的标准。而监控信息的展示（可视化）也没有比较标准的和开源整合的方案。
- 目标：
  - 指标统计标准化、完善，可以对外提供完善的 metrics 的功能
  - 监控 API 标准化，同时可以适配常用的开源方案（如 Prometheus）
  - 监控信息的可视化与开源方案适配（如 Grafana）

## Reactive 支持

- Milestone: 0.8.0
- Motivation: 响应式编程是 Java 社区未来的发展趋势。越来越多的人开始用 RxJava 1.x/2.x，Java 9 也将 Reactive Streams 的接口引入了 JDK，Spring 5.0 也引入了 Spring WebFlux / Project Reactor。可以说后续 reactive 是发展趋势。
- 目标：支持 Reactive 的形式（包括指标统计的实现，限流、降级、负载等算子的实现）

## 更多适配（生态）

- Milestone: 流动进行
- 复杂度：低
- Motivation：与更多的主流框架适配，与更多的动态配置/存储适配，可以吸引更多的用户。
- 目标：
  - 框架适配：如更完善的 gRPC 适配、RxJava 适配等
  - DataSource 适配：如 etcd, ZooKeeper, Git, MongoDB, MySQL 等
- Note：欢迎大家来贡献

## 高版本 Java 适配

- Milestone: 0.4.0（流动进行）
- Motivation：Java 现在迅猛发展，JDK 11 都快 GA 了。虽然大部分开发者仍在使用 Java 8，但是 JDK 发布下一个 LTS 版本后，预计逐渐会有一些开发者开始用新的 LTS 版本。因此对高版本的 Java 适配可以提上日程了。
- 主要关注点：JDK 9 引入的模块化可能会有比较大的影响。后续可以先在 CI 上开启高版本 JDK 的构建，看看有什么问题。

## 智能化

- Milestone: 1.1.0（长期进行）
- Motivation：当前用户需要根据系统的情况自己评估出一些数据来设定规则。如果能够比较好地利用历史的指标统计数据，根据设计的算法去动态地去调配规则，对用户来说是非常有吸引力的。
