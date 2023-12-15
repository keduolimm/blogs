

# 概念
- G
  - Goutine
- M
  - Machine内核线程
- P
  - Processor

主要是用户态的协程 和 内核态的线程 对应关系

N:1 模型， 就是coroutine, 

非抢占式的协作，需要主动放弃CPU, 其他协程才能被调用到

1:1 模式，没用充分利用资源，切换有代价

M:N 模式

会有一个协程调度器
局部性会差，

引入Processor




