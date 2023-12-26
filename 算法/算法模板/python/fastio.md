
# 前言

这边讲述python相关的IO读写

---

# 一般读

```python []
# 单行一变量
m = int(input())

# 多变量一行
n, k, x = list(map(int, input().split()))

# 数组读
arr = list(map(int, input().split()))
```

# 快读

还是需要引入buffer

```python
RI = lambda: map(int, sys.stdin.buffer.readline().split())
RS = lambda: map(bytes.decode, sys.stdin.buffer.readline().strip().split())
RILST = lambda: list(RI())
```