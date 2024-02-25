
# 牛客周赛 Round 33 解题报告 | 珂学家 | 思维场

---
# 前言

---
# 整体评价

感觉这场更偏思维，F题毫无思路，但是可以模拟骗点分, E题是dij最短路.

---

## [A. 小红的单词整理](https://ac.nowcoder.com/acm/contest/75630/A)

类型: 签到

```python3 []
w1,w2 = input().split()
print (w2)
print (w1)
```

---
## [B. 小红煮汤圆](https://ac.nowcoder.com/acm/contest/75630/B)

思路: 模拟

可以从拆包的角度去构建模拟

> 注意拆一包，可以烧好几次的情况

```python3 []
n, x, k = list(map(int, input().split()))

res = []
pack = 0
left = 0
for i in range(n):
    left += x
    pack += 1
    while left >= k:
        res.append(pack)
        left -= k
        pack = 0

print (len(res))
print (*res, sep=' ')
```

---
## [C. 小红的 01 串](https://ac.nowcoder.com/acm/contest/75630/C)

思路: 枚举

其实只能删除第1/第2，只能保证删除数组的前面一个区间，最多保留一个

题外话

如果题目改成第1/第2/.../第k，那最大价值数组是多少？

其实也是类似的思路

---

枚举最后一个被删除的位子

这样价值为 [前缀最多一位] + [后缀区间价值和]

```python3 []
s = input()

n = len(s)

n0, n1 = 0, 0
for c in s:
    if c == '0':
        n0 += 1
    else:
        n1 += 1

# 然后枚举
res = max(0, n1 - n0)

t1 = 0
for i in range(n):
    if s[i] == '0':
        n0 -= 1
    else:
        n1 -= 1
        t1 += 1
    res = max(res, n1 - n0 + (1 if t1 > 0 else 0))
    
print (res)
```

---
## [D. 小红的数组清空](https://ac.nowcoder.com/acm/contest/75630/D)

思路: 贪心

尽量从最小值开始删除，带动连续数组

貌似这边for嵌套while，看似时间复杂度很高，但实际上，每次都会有效删除至少1，而n最多1e5，所以均摊下来为O(1)

时间复杂度为$O(n)$


```python3 []
# 贪心
n = int(input())
arr = list(map(int, input().split()))

from collections import Counter

cnt = Counter(arr)

brr = [[k, v] for (k, v) in cnt.items()]
brr.sort(key=lambda x: x[0])

ans = 0
m = len(brr)
for i in range(m):
    if brr[i][1] > 0:
        j = i + 1
        cost = brr[i][1]
        brr[i][1] = 0
        ans += cost
        while j < m and cost > 0 and brr[j - 1][0] + 1 == brr[j][0]:
            cost = min(cost, brr[j][1])            
            brr[j][1] -= cost
            j += 1

print (ans)
```

---
## [E. 小红勇闯地下城](https://ac.nowcoder.com/acm/contest/75630/E)

思路: dijkstra最短路

题型: 板子题

```python3 []
t = int(input())

import heapq
from math import inf

def solve(grids, health):
    h, w = len(grids), len(grids[0])
    
    sy, sx = -1, -1
    ty, tx = -1, -1
    for i in range(h):
        for j in range(w):
            if grids[i][j] == 'S':
                sy, sx = i, j
            elif grids[i][j] == 'T':
                ty, tx = i, j
                
    if sy == -1 or ty == -1:
        return "No"
    arr = []
    
    vis = [[inf] * w for _ in range(h)]
    heapq.heappush(arr, (0, sy, sx))
    vis[sy][sx] = 0
    
    while len(arr) > 0:
        cur = heapq.heappop(arr)
        if cur[0] > vis[cur[1]][cur[2]]:
            continue
        if cur[0] >= health:
            continue
        if cur[1] == ty and cur[2] == tx:
            return "Yes"
        for (dy, dx) in [(1, 0), (-1, 0), (0, -1), (0, 1)]:
            gy = cur[1] + dy
            gx = cur[2] + dx
            if gy >= h or gy < 0 or gx >= w or gx < 0:
                continue
            
            tcost = 0
            if grids[gy][gx] >= '0' and grids[gy][gx] <= '9':
                tcost = ord(grids[gy][gx]) - ord('0')
                
            if vis[gy][gx] > cur[0] + tcost and cur[0] + tcost < health:
                vis[gy][gx] = cur[0] + tcost
                heapq.heappush(arr, (vis[gy][gx], gy, gx))
    
    return "No"

for _ in range(t):
    n, m, h = list(map(int, input().split()))
    grids = []
    for _ in range(n):
        grids.append(input())
    print (solve(grids, h))
```

---
## [F. 小红的数组操作](https://ac.nowcoder.com/acm/contest/75630/F)

后期再补上

---
# 写在最后
![alt](https://uploadfiles.nowcoder.com/images/20240218/446702330_1708258983710/D2B5CA33BD970F64A6301FA75AE2EB22)

