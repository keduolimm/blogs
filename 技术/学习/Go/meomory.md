
# go的内存结构

- mheap
- mcache
- mspan

mspan是page的整数倍，是go内存分配的基本单元

mcache其实和P相关, Processor

mheap内部划分为0~67(68个)根据大小划分的块

这样可以大大减少内存malloc的碎片，mspan内部依旧存在

主要是尽量无锁操作

