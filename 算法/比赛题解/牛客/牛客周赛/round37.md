
# 牛客周赛 Round 37 解题报告 | 珂学家 | AK场

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20240317/446702330_1710683147219/D2B5CA33BD970F64A6301FA75AE2EB22)

---

# 整体评价

有幸参加了内测，感觉最难的C和D，这两题偏思维，E/F偏板子和套路。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 雾之湖的冰精](https://ac.nowcoder.com/acm/contest/77231/A)

签到

```python3 []
a, b = list(map(int, input().split()))
 
if a + b > 9:
    print ("No")
else:
    print ("Yes")
```
---

## [B. 博丽神社的巫女](https://ac.nowcoder.com/acm/contest/77231/B)

思路: 找边界

其实排序后，二分更好

```python3 []
n, m = list(map(int, input().split()))
 
arr = list(map(int, input().split()))
arr.sort()
 
pos = -1
for i in range(n):
    if m < arr[i]:
        break
    pos = i
 
if pos == -1:
    print (0, m)
else:
    print (pos + 1, m - arr[pos])
```

---

## [C. 红魔馆的馆主](https://ac.nowcoder.com/acm/contest/77231/C)

这题思路和解法蛮多的，这边介绍一种。

### 方法一: 找上界 + 枚举

对于一个数n, 可以乘以1000，然后余495，这样就得到一个值x

然后找到一个y，使得 x+y = 495，即末尾添加y(不足三位，前面补齐0)

就可以找到一个上界，即最多添加3位。

有了最多3位的结论后，

只需要从0枚举到999，然后测试即可。

需要特别注意前置0的情况

```python3 []
p = 495

def solve():
    n = int(input())
    if n % p == 0:
        return -1
    sn = str(n)
    # 枚举0到999
    for i in range(10):
        v = int(sn + str(i))
        if v % p == 0:
            return i
    for i in range(100):
        v = int(sn + ("%02d" % i))
        if v % p == 0:
            return ("%02d" % i)
    for i in range(0, 1000):
        v = int(sn + ("%03d" % i))
        if v % p == 0:
            return ("%03d" % i)
    # unreachable
    return "000"

print (solve())
```

---

### 方法二: 迭代加深DFS

我一开始用了这个解法

```python3 []
p = 495
def dfs(v, s, d):
    if d == 0:
        return s if int(str(v) + s) % p == 0 else None
    for c in "0123456789":
        r = dfs(v, s+c, d - 1)
        if r:
            return r
    return None

def solve(n, maxDepth):
    if n % p == 0:
        return (-1)
    # 迭代加深
    for i in range(1, maxDepth+1):
        r = dfs(n, "", i)
        if r:
            return r
    return ""
        
n = int(input())
print (solve(n, 3))
```


---

## [D. 迷途之家的大贤者](https://ac.nowcoder.com/acm/contest/77231/D)

思路: 博弈 + 剪枝

经典的最大最小剪枝写法

```python []
def maximize(ss):
    l = len(ss)
    ans = minimize(ss)
    for i in range(l):
        for j in range(i, l):
            ts = ss[:i] + ss[j+1:]
            if ts == "":
                continue
            tmp = minimize(ts)
            if ans < tmp:
                ans = tmp
    return ans

def minimize(ss):
    l = len(ss)
    ans = ss
    for i in range(l):
        # 加个剪枝
        if ans <= ss[:i]: 
            continue
        for j in range(i, l):
            ts = ss[:i] + ss[j+1:]
            if ts == "":
                continue
            if ans > ts:
                ans = ts
    return ans

n = int(input())
print (maximize(input()))
```

但是这样时间复杂度为$O(n^4)$

---

最好的解为

纳什均衡，第一个和最后一个的最大值

```python3 []
n = int(input())
s = input()
print (max(s[0], s[-1]))
```

---
## [E. 魔法之森的蘑菇](https://ac.nowcoder.com/acm/contest/77231/E)

思路: 状态设计 + BFS

很考验基本功，码量也是最大的一个

在二维矩阵的基础上，引入一个方向位d(上下左右)

这样构建了状态 dp[y][x][d], 跑一遍BFS即可。

```python3 []

from math import inf
from collections import deque

t = int(input())
for _ in range(t):
    h, w = list(map(int, input().split()))
    g = []
    
    sy, sx = 0, 0
    ty, tx = 0, 0
    for i in range(h):
        s = input()
        g.append(s)
        for j in range(w):
            if s[j] == 'S':
                sy, sx = i, j
            elif s[j] == 'T':
                ty, tx = i, j                
    
    vis = [ [ [inf] * 4 for _ in range(w) ] for _ in range(h) ]
    
    deq = deque()
    for i in range(4):
        deq.append((sy, sx, i))
        vis[sy][sx][i] = 0
    
    dirs = [(1, 0), (0, -1), (0, 1), (-1, 0)]
    while len(deq) > 0:
        y, x, d = deq.popleft()
        
        y2 = y + dirs[d][0]
        x2 = x + dirs[d][1]
        if 0 <= y2 < h and 0 <= x2 < w:
            if g[y2][x2] == '#':
                continue
            
            if g[y2][x2] != '*':
                if vis[y2][x2][d] > vis[y][x][d] + 1:
                    deq.append((y2, x2, d))
                    vis[y2][x2][d] = vis[y][x][d] + 1
            else:
                # 核心的逻辑就在这
                for j in range(4):
                    if j != 3 - d:
                        if vis[y2][x2][j] > vis[y][x][d] + 1:
                            deq.append((y2, x2, j))
                            vis[y2][x2][j] = vis[y][x][d] + 1
    
    r = min(vis[ty][tx])
    print (-1 if r == inf else r)
```
---

## [F. 三途川的摆渡人](https://ac.nowcoder.com/acm/contest/77231/F)

思路: 状压DP

这题还是放水了，如果把值域放大，就很麻烦了。

令dp[i][j] 为前i项，其与值为j的最大删除个数

转移方程

sj = j & arr[i+1]

dp[i + 1][sj] = max(dp[i][sj] + 1, dp[i][j])

然后用滚动数组，把第一维度优化掉

```python3 []
from math import inf

t = int(input())
for _ in range(t):
    n = int(input())
    arr = list(map(int, input().split()))
    dp = [-inf] * 256
    dp[255] = 0
    
    for v in arr:
        dp2 = [-inf] * 256
        for i in range(256):
            dp2[i] = max(dp2[i], dp[i] + 1)
            dp2[i & v] = max(dp2[i & v], dp[i])
            
        dp = dp2
    print (-1 if dp[0] == -inf else dp[0])
```

这样时间复杂度为 $O(n*m)$

针对该数据，还可以继续优化，因为200个桶，2*10^5的数据，这样可以把数据压缩为200个(过滤掉重复的)

这样优化到$O(min(n, 200) * m)$

---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240317/446702330_1710683066947/D2B5CA33BD970F64A6301FA75AE2EB22)






