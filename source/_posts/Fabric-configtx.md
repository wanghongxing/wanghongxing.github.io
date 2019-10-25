---
title: Fabric configtx
date: 2019-08-27 14:08:56
tags:
  - fabric
  - blockchain
---

configtx.yaml 文件 定义了组织、order创世块 profile、通道profile。
通过order创世块profile可以生成order的创世块，也就是整个网络的创世块，其中定义了联盟成元组织。
通过通道profile 定义了联盟的通道profile，通过这个profile可以生成创建通道的tx，以及每个组织的anchor成员的tx。
所谓的tx就是通过tx来提交一个操作，比如：
创建通道的tx可以提交给order创建一个通道；好像后续可以update更新什么的。
组织成员anchor的tx，可以定义一个组织的anchor成员。

创建通道的时候提交tx，会生成一个通道的创世块，缺省文件名就是通道名称.block；
其它节点加入通道需要使用通道的创世块。