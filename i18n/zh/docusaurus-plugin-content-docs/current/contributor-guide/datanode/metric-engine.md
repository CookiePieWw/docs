---
keywords: [Metric 引擎, 逻辑表, 物理表, DDL 操作]
description: 介绍了 Metric 引擎的概念、架构及设计，重点描述了逻辑表与物理表的区别和批量 DDL 操作的实现。
---

# Metric 引擎

## 概述

`Metric` 引擎是 GreptimeDB 的一个组件，属于存储引擎的一种实现，主要针对可观测 metrics 等存在大量小表的场景。

它的主要特点是利用合成的物理宽表来存储大量的小表数据，实现相同列复用和元数据复用等效果，从而达到减少小表的存储开销以及提高列式压缩效率等目标。表这一概念在 `Metric` 引擎下变得更更加轻量。

## 概念

`Metric` 引擎引入了两个新的概念，分别是逻辑表与物理表。从用户视角看，逻辑表与普通表完全一样。从存储视角看，物理 Region 就是一个普通的 Region。

### 逻辑表
逻辑表，即用户定义的表。与普通的表都完全一样，逻辑表的定义包括表的名称、列的定义、索引的定义等。用户的查询、写入等操作都是基于逻辑表进行的。用户在使用过程中不需要关心逻辑表和普通表的区别。

从实现层面来说，逻辑表是一个虚拟的表，它并不直接读写物理的数据，而是通过将读写请求映射成对应物理表的请求来实现数据的存储与查询。

### 物理表
物理表是真实存储数据的表，它拥有若干个由分区规则定义的物理 Region。

## 架构及设计

`Metric` 引擎的主要设计架构如下：

![Arch](/metric-engine-arch.png)

在目前版本的实现中，`Metric` 引擎复用了 `Mito` 引擎来实现物理数据的存储及查询能力，并在此之上同时提供物理表与逻辑表的访问能力。

在分区方面，逻辑表拥有与物理表完全一致的分区规则及 Region 分布。这是非常自然的，因为逻辑表的数据直接存储在物理表中，所以分区规则也是一致的。

在路由元数据方面，逻辑表的路由地址为逻辑地址，即该逻辑表所对应的物理表是什么，而后通过该物理表进行二次路由取得真正的物理地址。这一间接路由方式能够显著减少 `Metric` 引擎的 Region 发生迁移调度时所需要修改的元数据数量。

在操作方面，`Metric` 引擎仅对物理表的操作进行了有限的支持以防止误操作，例如通过禁止写入物理表、删除物理表等操作防止影响用户逻辑表的数据。总体上可以认为物理表是对用户只读的。

为了提升对大量表同时进行 DDL（Data Definition Language，数据操作语言）操作时性能，如 Prometheus Remote Write 冷启动时大量 metrics 带来的自动建表请求，以及前面提到的迁移物理 Region 时大量路由表的修改请求等，`Metric` 引擎引入了一些批量 DDL 操作。这些批量 DDL 操作能够将大量的 DDL 操作合并成一个请求，从而减少了元数据的查询及修改次数，提升了性能。

除了物理表的物理数据 Region 之外，`Metric` 引擎还额外为每一个物理数据 Region 创建了一个物理的元数据 Region，用于存储 `Metric` 引擎自身为了维护映射等状态所需要的一些元数据。这些元数据包括逻辑表与物理表的映射关系，逻辑列与物理列的映射关系等等。
