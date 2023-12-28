
# 前言

这部分内容，参考了小林的技术blog。

---
# Redis多线程


--- 

# 数据类型

- string
- list
- hash
- set
- zset

后续添加一些高阶数据结构

- GEO
  - 支持GeoHash地理，近距离查询
- hyperloglog
- Stream

    - 全局分配的id，同时引入消费组的概念
- Bitmap

  - 比如bloomfilter

---




---
# 持久化相关

- AOF
- RDB
- AOF & RDB 混合


AOF重写的概念

根据内存的key/value, 重新生成命令

COW(写时拷贝)


Q: 为啥是子进程，而不是线程

线程需要加锁，才能保证数据安全，子进程会引入COW机制，天然数据安全

此时如果有新的数据写入

写内存 -> AOF 缓存 -> AOF 重写缓存

---

# 主从复制


---

# 集群模式

- 简单主从模式
- 哨兵模式
- Redis Cluster

哈希槽 2^14 

----






