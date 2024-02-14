

# 牛客周赛 Round 32 解题报告 | 珂学家 | 状压 + 前缀和&异或map技巧

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240212/446702330_1707716767128/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

属于补题，大致看了下，题都很典。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)


---

## [A. 小红的 01 背包](https://ac.nowcoder.com/acm/contest/75174/A)

思路: 数学题

```python3 []
v, x, y = list(map(int, input().split()))

print (v // x * y)
```

---

## [B. 小红的 dfs](https://ac.nowcoder.com/acm/contest/75174/B)

思路: 枚举

其实横竖都有dfs字符，只有3种情况

- 第一行，第一列为dfs
- 第二行，第二列为dfs
- 第三行，第三列为dfs

枚举取最小代价即可

```python3 []
grids = []
for i in range(3):
    grids.append(input())
    
# 枚举
def cost(y, x, ch) -> int:
    if grids[y][x] == ch:
        return 0
    return 1

r1 = cost(0, 0, 'd') + cost(0, 1, 'f') + cost(0, 2, 's') \
    + cost(1, 0, 'f') + cost(2, 0, 's')

r2 = cost(1, 0, 'd') + cost(1, 1, 'f') + cost(1, 2, 's') \
    + cost(0, 1, 'd') + cost(2, 1, 's')

r3 = cost(2, 0, 'd') + cost(2, 1, 'f') + cost(2, 2, 's') \
    + cost(0, 2, 'd') + cost(1, 2, 'f')


print (min(r1, r2, r3))
```
---

## [C. 小红的排列生成](https://ac.nowcoder.com/acm/contest/75174/C)

思路: 贪心

猜结论，感觉就是排序后

累加 abs(i - arr[i])

```python3 []
n = int(input())

arr = list(map(int, input().split()))
arr.sort()

res = 0
for i in range(n):
    res += abs(i + 1 - arr[i])

print (res)
```

---

## [D. 小红的二进制树](https://ac.nowcoder.com/acm/contest/75174/D)

思路: 树形DP

自底向上的DFS即可


```python3 []
n = int(input())
s = input()

g = [[] for _ in range(n + 1)]

for i in range(n - 1):
    u, v = list(map(int, input().split()))
    g[u].append(v)
    g[v].append(u)
    
from types import GeneratorType

def bootstrap(f, stack=[]):
    def wrappedfunc(*args, **kwargs):
        if stack:
            return f(*args, **kwargs)
        else:
            to = f(*args, **kwargs)
            while True:
                if type(to) is GeneratorType:
                    stack.append(to)
                    to = next(to)
                else:
                    stack.pop()
                    if not stack:
                        break
                    to = stack[-1].send(to)
            return to

    return wrappedfunc

    
# python dfs会栈溢出
dp = [0] * (n + 1)

@bootstrap
def dfs(u: int, fa: int):
    for v in g[u]:
        if v == fa:
            continue
        yield dfs(v, u)
        dp[u] += dp[v]
    if s[u - 1] == '1':
        dp[u] += 1
    yield

dfs(1, -1)

for i in range(1, n + 1):
    print (dp[i] - (1 if s[i - 1] == '1' else 0))
```

---

## [E. 小红的回文数](https://ac.nowcoder.com/acm/contest/75174/E)

思路: 前缀和 + 异或map技巧

就是0~9这10个构建一个字符集

由于奇偶特性可以借助异或来表达

这样就变成1024种状态

时间复杂度为

$O(10 * n)$

```python3 []
x = input()

from collections import Counter

cnt = Counter()
cnt[0] = 1

res = 0
s = 0
for c in x:
    p = ord(c) - ord('0')
    s ^= (1 << p)
    res += cnt[s]
    for i in range(10):
        res += cnt[s ^ (1 << i)]
    cnt[s] += 1
    
print (res)
```

----

## [F. 小红的矩阵修改](https://ac.nowcoder.com/acm/contest/75174/F)

思路: 状压 + 3进制

很典的一道状压入门题

时间复杂度:

$O(3^{2n} * m)$

```python3 []
n, m = list(map(int, input().split()))

def mapping(c) -> int: 
    if c == 'r': 
        return 0
    elif c == 'e':
        return 1
    return 2

grids = []
for i in range(n):
    s = input()
    grids.append([mapping(c) for c in s])

# 状压DP
# 3进制

from math import pow, inf
y = int(pow(3, n))

dp = [0] * y

def compute(idx, state) -> int:
    if not isValid(state):
        return inf
    
    cost = 0
    for i in range(n):
        d = state % 3
        state = state // 3
        if grids[i][idx] != d:
            cost += 1
    return cost

def isValid(state) -> bool:
    r = []
    for i in range(n):
        r.append(state % 3)
        state = state // 3
    for i in range(len(r) - 1):
        if r[i] == r[i+1]:
            return False
    return True

def twoValid(s1: int, s2: int) -> bool:
    r1, r2 = [], []
    for i in range(n):
        r1.append(s1 % 3)
        s1 = s1 // 3
        r2.append(s2 % 3)
        s2 = s2 // 3
    for i in range(len(r1) - 1):
        if r1[i] == r1[i+1] or r2[i] == r2[i+1]:
            return False
    for i in range(len(r1)):
        if r1[i] == r2[i]:
            return False
    return True
            
for i in range(y):
    dp[i] = compute(0, i)
    

for j in range(1, m):
    dp2 = [inf] * y
    for k in range(y):
        c = compute(j, k)
        for k2 in range(y):
            if twoValid(k, k2):
                dp2[k] = min(dp2[k], dp[k2] + c)
    dp = dp2

print(min(dp))    
```

---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20240212/446702330_1707716691901/D2B5CA33BD970F64A6301FA75AE2EB22)


