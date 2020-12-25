---
layout: post
title: redis cluster pipeline
---

## redis cluster
从3.0开始，redis开始支持集群模式。

![cluster]({{"/assets/img/redis-cluster.png"}})

- 无中心架构

- 按key将数据哈希到16384个slot上
		HASH_SLOT = CRC16(key) mod 16384
- 集群中的不同节点分别负责一部分slot

## redis pipeline



## 代码


