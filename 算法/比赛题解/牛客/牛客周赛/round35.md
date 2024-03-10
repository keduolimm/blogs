
# 牛客周赛 Round 35 解题报告 | 珂学家 | 构造 + 组合数学

--- 
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240303/446702330_1709473997623/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价
F/G是数学题，E是一道有趣的构造题, 需要一点点空间想象力，其他几题也不错。不过整场被python的库函数，折磨得崩溃，T_T.

---

## [A. 小红的字符串切割](https://ac.nowcoder.com/acm/contest/76133/A)

题型: 签到

```python3 []
s = input()
half = len(s) // 2

print (s[0:half])
print (s[half:])
```
---
## [B. 小红的数组分配](https://ac.nowcoder.com/acm/contest/76133/B)

思路: map应用+构造

map的一个应用，偶数肯定ok，奇数肯定没法构造

```python3 []
from collections import Counter
n = int(input())

arr = list(map(int, input().split()))

cnt = Counter(arr)

arr1 = []
arr2 = []
ok = True
for (k, v) in cnt.items():
    if v % 2 == 1:
        ok = False
    else:
        arr1.extend([k] * (v // 2))
        arr2.extend([k] * (v // 2))

if ok:
    print (*arr1, sep = ' ')
    print (*arr2, sep = ' ')
else:
    print (-1)
```

---

## [C. 小红关鸡](https://ac.nowcoder.com/acm/contest/76133/C)

思路: 双指针

排序后，双指针模拟，枚举然后取最大的覆盖点集数

```python3 []
n, k = list(map(int, input().split()))
arr = list(map(int, input().split()))

res = 0.0

arr.sort()
j = 0
for i in range(n):
    while j < n and arr[j] - arr[i] <= k:
        j += 1
    d = j - i
    res = max(res, d * 1.0 / n)

print ("%.6f" % (res))
```

---

## [D. 小红的排列构造](https://ac.nowcoder.com/acm/contest/76133/D)

思路: 模拟

把多的索引位置组成一个序列，然后把少的数字组成一个序列

然后zip一下，就出来了

```python3 []
n = int(input())
arr = list(map(int, input().split()))

vis = [False] * (n + 1)

more = []
for (i, v) in enumerate(arr):
    if v > n or vis[v]:
        more.append(i + 1)
    else:
        vis[v] = True

lack = []
for i in range(1, n + 1):
    if not vis[i]:
        lack.append(i)

print (len(more))
for (k, v) in zip(more, lack):
    print (k, v)
```

---
## [E. 小红的无向图构造](https://ac.nowcoder.com/acm/contest/76133/E)

思路: 构造

- 先构建树结构，保证最短距离满足要求
- 在满足边数

前一个相对简单，后一个需要一点点空间想象力

1. 可以在同距离的点集合中，构建星状结构的边

2. 在距离差1的两个点集合中，交叉构建边

这两种边，对之前点的最短距离，毫无影响


```python3 []
n, m = list(map(int, input().split()))
arr = list(map(int, input().split()))

def solve(res):
    hp = set()
    def add(v1, v2):
        if v1 > v2:
            v1, v2 = v2, v1
        hp.add((v1, v2))
        res.append((v1, v2))
        
    def exist(v1, v2):
        if v1 > v2:
            v1, v2 = v2, v1
        return (v1, v2) in hp
        
    # 1. 满足点的距离
    tx = [(i, v) for (i, v) in enumerate(arr) if i != 0]
    tx.sort(key=lambda x: x[1])
    
    path = [[] for _ in range(n)]
    path[0].append(0)
    acc = 0
    far = 0
    for (p, v) in tx:
        if v <= far + 1:
            path[v].append(p)
            add(path[v - 1][-1], p)
            acc += 1
        else:
            return False
        far = max(v, far)
    
    # 判定所需的边数是否大于要求
    if acc > m:
        return False
    
    # 2. 补充边的阶段
    #    2.1. 补充同距离的边
    j = 0
    while acc < m and j < n:
        brr = path[j]
        bn = len(brr)
        for si in range(bn):
            for sj in range(si + 1, bn):
                if acc == m: 
                    return True
                add(brr[si], brr[sj])
                acc += 1
        j += 1
    
    #    2.2. 补充距离为1的点集合之间的边
    for i in range(n - 1):
        for e1 in path[i]:
            for e2 in path[i + 1]:
                if acc == m:
                    return True
                if not exist(e1, e2):
                    add(e1, e2)
                    acc += 1
    return acc == m

ls = []
if solve(ls):
    for (p1, p2) in ls:
        print (p1 + 1, p2 + 1)
else:
    print (-1)
```

---
## [F. 小红的子序列权值和（easy）](https://ac.nowcoder.com/acm/contest/76133/F)

和G题，一起讲

---

## [G. 小红的子序列权值和（hard）](https://ac.nowcoder.com/acm/contest/76133/G)

思路: 经过公式的抽象提炼，可以归纳为 加权和 乘积

其实1的情况比较特殊

![alt](https://uploadfiles.nowcoder.com/images/20240303/446702330_1709473242288/D2B5CA33BD970F64A6301FA75AE2EB22)


```python3 []
def ksm(b, v):
    r = 1
    while v > 0:
        if v % 2 == 1:
            r = r * b % mod
        v //= 2
        b = b * b % mod
    return r

class Factorial:
    def __init__(self, N, mod) -> None:
        N += 1
        self.mod = mod
        self.f = [1 for _ in range(N)]
        self.g = [1 for _ in range(N)]
        for i in range(1, N):
            self.f[i] = self.f[i - 1] * i % self.mod
        self.g[-1] = pow(self.f[-1], mod - 2, mod)
        for i in range(N - 2, -1, -1):
            self.g[i] = self.g[i + 1] * (i + 1) % self.mod
 
    def fac(self, n):
        return self.f[n]
 
    def fac_inv(self, n):
        return self.g[n]
 
    def comb(self, n, m):
        if n < m or m < 0 or n < 0: return 0
        return self.f[n] * self.g[m] % self.mod * self.g[n - m] % self.mod
 
    def perm(self, n, m):
        if n < m or m < 0 or n < 0: return 0
        return self.f[n] * self.g[n - m] % self.mod
 
    def catalan(self, n):
        return (self.comb(2 * n, n) - self.comb(2 * n, n - 1)) % self.mod
 
    def inv(self, n):
        return self.f[n - 1] * self.g[n] % self.mod

n = int(input())
arr = list(map(int, input().split()))

mod = 10 ** 9 + 7
n1, n2, n3 = arr.count(1), arr.count(2), arr.count(3)

fac = Factorial(n, mod)
    
s1 = ksm(2, n1)
s2 = sum([fac.comb(n2, i) * (i + 1) % mod for i in range(n2 + 1)])
s3 = sum([fac.comb(n3, i) * (i + 1) % mod for i in range(n3 + 1)])

res = s1 * s2 * s3 % mod
res = (res - 1 + mod) % mod 

print (res)
```

---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20240303/446702330_1709474009070/D2B5CA33BD970F64A6301FA75AE2EB22)

