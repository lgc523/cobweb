---
title: "Metrics、Tracing、 Logging"
date: 2022-02-10T00:04:10+08:00
draft: true
toc: true
tags: 
  - distributed
---

## 分布式的基石 --- 可观测性

可观测性(Observability) 和可控制性(Controllability) 是由匈牙利数学家 Rudolf E·Kålmån 针对线性动态控制系统提出的一组对偶属性，原本的含义是 **可以由其外部输出推断其内部状态的程度**。

> Peter Bourgon **metric-tracing-and-logging**
>
> https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html



![metrics-tracing-logging](https://s2.loli.net/2022/02/10/8NBKUCrGp6aIjiM.png)

## logging

日志的职责是**记录系统运行期间发生过的离散事件**，通过记录**分析程序行为**。

### 事件日志

日志应该做到像流水账一样，格式统一，内容恰当，没有遗漏，日志内容不该出现的内容不要有，该有的不少。

- **避免打印敏感信息**
- **避免引用慢操作**
  - 关注测试调试数据和实际生产数据的差别
- **避免打印追踪诊断信息** 
  - 不要打印方法输入参数、输出结果、方法执行时长的调试信息
- **避免误导他人**
  - 异常处理后不要打印堆栈
- **请求携带 TraceId**
  - 请求应该必须始终附带 TraceId (随请求响应返回到客户端)，如果出现异常，能够快读找到与问题相关的日志。
- **系统运行过程中的关键事件**
- **启动时输出配置信息**
  - 系统启动时或检测到配置更新，应将非敏感的配置信息输出
  - 连接的数据库、临时目录路径
  - 针对只执行一次或者少次的逻辑

### 收集缓冲、加工聚合、存储查询

涉及到 Kafka、es、logstash、beats。

加工聚合的方案中 依赖 es 的实时聚合统计用于 **即席查询**，另外通过 logstash 中的聚合插件在收集日志后自动生成某些常用的、固定的聚合指标用于**固定查询**。

日志数据随着时间的推移会逐渐失去价值，可以区分出冷数据和热数据，进而采用不同的硬件策略存储。

## tracing

单体系统追踪的范畴基本只局限于栈追踪，微服务追踪调用轨迹跨越多个服务，追踪包括服务间网络传输和服务内部的调用堆栈信息。

追踪的目的主要是**排查故障**，分析调用链的哪一部分、哪个方法出现错误或阻塞，输入和输出是否符合预期。



> https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36356.pdf
>
> **Dapper, a Large-Scale Distributed Systems Tracing Infrastructure**

2010 年 Google 发表的论文提出了 **追踪(Traec)** 与 **跨度(Span)**概念来进行有效的分布式追踪。

### 追踪与跨度

从客户端发起请求抵达系统的边界开始，记录请求流经的每一个服务，**直到向客户端返回响应为止**，整个过程就称为一次**追踪**。

每次开始调用服务前都需要先埋入一个调用记录，这个记录称为一个**跨度**，Span 的数据结构应足够简洁、完备。

每一次 Trace 实际上都是由若干个**有顺序、有层级关系**的 Span 组成的一棵**追踪树**。

![trace-tree](https://s2.loli.net/2022/02/11/c68MnvQBfKT37is.png)

**微服务可能是异构语言、协议来进行交互，所以追踪系统需要增加功能应对多种语言和协议。**

- **低性能损耗**
  - 越慢的服务越需要追踪，不能对服务本身产生明显的性能负担
- **对应用透明**
  - 尽量以非侵入或者少侵入的方式来实现追踪
- **随应用扩缩**
  - 追踪系统跟随系统扩缩
- **持续的监控**
  - 持续监控，防止偶尔的抖动

### 数据收集

追踪系统根据数据收集方式的差异，可分为三种主流的实现方式。

- **基于日志的追踪 Log-Based Tracing**
  - 将 Trace、Span 等信息直接输出到日志中
  - 所有节点的日志归集到一起
  - 从全局日志中反推出完整的调用链拓扑关系
  - 对应用侵入和性能影响都较低，但是依赖日志归集过程，不够精准，可能会出现延迟和缺失
  - 代表的有 Spring Cloud Sleuth，SofaTracing
- **基于服务的追踪 Service-Based Tracing**
  - 通过给目标应用注入追踪探针
  - 从目标系统中监控到的服务调用信息，通过请求发送给追踪系统
  - 消耗资源较多，侵入更多，但是追踪精较确、精准
- **基于边车代理的追踪 Sidecar-Based Tracing**
  - Service mesh 专属方案，目前最理想的方案，Envoy
  - 对应用透明，了解到的大概是承接了网络流量进行处理，**但是只能实现服务调用层面的追踪**
  - 通过控制平面上报，保证了最佳的精确性

### 追踪规范化

OpenTelemetry

## metrics

度量是指**对系统中某一类信息的统计聚合**，主要目的是**监控**和**预警**，**在度量的指标达到风险阈值时触发事件**。



## 
