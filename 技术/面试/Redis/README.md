
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

# 缓存雪崩/击穿/穿透

定义

1. 缓存雪崩

多个key(热点)同时过期，导致请求落到db上，进而引发系统的崩盘

解决方案

- 多个key添加随机扰动
- key永不过期

2. 缓存击穿

key过期了，请求同时落DB

解决方案

- 请求合并
- 分布式锁
- key永不过期

3. 缓存穿透

访问的资源不在redis/db上

解决方案

- 引入空对象缓存
- 引入布隆过滤器

---

# 过期key清理策略


- lazy删除
- 定期删除

redis会单独维护设置expire的key列表

然后定期，随机抽取一部分，去去除过期的key

这边需要配置一些参数，来适配不同的场景






