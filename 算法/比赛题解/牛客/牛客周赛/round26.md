
# 牛客周赛 Round 26 解题报告 | 珂学家 | 0-1 BFS + 状态机DP


---

# 前言

---

# 整体评价

T3是一道0-1 BFS题, 这样时间复杂度可以控制在O(n*m), 也可以用优先队列。

T4这类题型，在牛客Round周赛系列出现好多次了，要么状态机DP，要么容斥，如果n很大，就用矩阵幂优化。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的整数操作](https://ac.nowcoder.com/acm/contest/72779/A)

思路：同余分组

对k进行取模分组，同余的任意两个数，一定可以构造成一样

```python []
from collections import Counter

n, k = list(map(int, input().split()))
arr = list(map(int, input().split()))

cnt = Counter()
for v in arr:
    cnt[v % k] += 1

res = max(cnt.values())

print (res)
```
---

## [B. 小红的01串](https://ac.nowcoder.com/acm/contest/72779/B)

思路：奇偶分析，找规律

可以枚举最终趋同的数是0，还是1

同时相邻2数翻转，其0,1本身个数的奇偶是不改变的

这样可以发现，如果0的个数，1的个数都是奇数，必然是死局，无法构造。

```python []
t = int(input())

while t > 0:
    t -= 1
    s = input()
    n0, n1 = 0, 0
    for c in s:
        if c == '0':
            n0 += 1
        else:
            n1 += 1
    if n0 % 2 == 1 and n1 % 2 == 1:
        print ("No")
    else:
        print ("Yes")
```

---
## [C. 小红闯沼泽地](https://ac.nowcoder.com/acm/contest/72779/C)

思路：0-1 BFS，当然用优先队列寻路，也是的可以

这边采用0-1 BFS，应该时间复杂度最低

```python []
from collections import deque
# from typings import inf

h, w = list(map(int, input().split()))

grids = []
for i in range(h):
    row = list(map(int, input().split()))
    grids.append(row)

deq = deque()
deq.append((0, 0))

dp = [[0x3f3f3f3f] * w for _ in range(h)]

# 0-1 bfs
# 或者优先队列

dp[0][0] = 0

dirs = [(1, 0), (0, -1), (0, 1)]

while len(deq) > 0:
    y, x = deq.popleft()
    for dy, dx in dirs:
        ty, tx = y + dy, x + dx
        if 0 <= ty < h and 0 <= tx < w:
            cost = 2 if grids[y][x] != grids[ty][tx] else 1
            if dp[ty][tx] > dp[y][x] + cost:
                dp[ty][tx] = dp[y][x] + cost
                if cost == 1:
                    deq.appendleft((ty, tx))
                else:
                    deq.append((ty, tx))

print (dp[h - 1][w - 1])
```

---
## [D. 小红的漂亮串（二）](https://ac.nowcoder.com/acm/contest/72779/D)

思路：状态机DP

引入7个状态

- any (0个red): 0
- r: 1
- re: 2
- red: 3
- r(带1个red): 4
- re(带1个red): 5
- red(带2个red): 6

因为n为10^6, 所以线性处理下就行了

```python []
mod = 10 ** 9 + 7

n = int(input())

# other: 0
# r: 1
# re: 2
# red: 3
# red, r: 4
# red, re: 5
# red, red: 6

# 状态机DP
dp = [25, 1, 0, 0, 0, 0, 0]

for i in range(1, n):
    dp2 = [0] * 7
    
    dp2[0] = (dp[0] * 25 + dp[1] * 24 + dp[2] * 24) % mod
    dp2[1] = (dp[0] + dp[1] + dp[2]) % mod
    dp2[2] = dp[1]
    
    dp2[3] = (dp[2] + dp[3] * 25 + dp[4] * 24 + dp[5] * 24) % mod
    dp2[4] = (dp[3] + dp[4] + dp[5]) % mod
    dp2[5] = dp[4]
    dp2[6] = (dp[5] + dp[6] * 26) % mod
    
    dp = dp2

print (dp[6])
```

---
# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20240101/446702330_1704106502167/D2B5CA33BD970F64A6301FA75AE2EB22)




