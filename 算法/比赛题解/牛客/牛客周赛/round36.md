
# 牛客周赛 Round 36 解题报告 | 珂学家 | 状态DP + 构造 + 9棵树状数组

---
# 前言

![alt](https://img-blog.csdnimg.cn/img_convert/dd8d0cf30723dc4e646da95dd9305cec.png)


---
# 整体评价

今天相对容易，E的构造题，感谢出题人极其善意的Case 1, 算是放水了。F题是个很典的结论题，由于存在动态点修改，所以引入树状数组做区间和的快速计算。

---
## [A. 小红的数位删除](https://ac.nowcoder.com/acm/contest/76609/A)

题型: 签到

```python3 []
s = input()

print (s[:-3])
```

---
## [B. 小红的小红矩阵构造](https://ac.nowcoder.com/acm/contest/76609/B)

思路: 模拟

```python3 []
h, w, s = list(map(int, input().split()))

ts = 0
grid = []
for i in range(h):
    arr = list(map(int, input().split()))
    grid.append(arr)
    ts += sum(arr) 

hs = set()
for i in range(h):
    v = 0
    for u in grid[i]:
        v = v ^ u
    hs.add(v)
for i in range(w):
    v = 0
    for j in range(h):
        v ^= grid[j][i]
    hs.add(v)
    
if len(hs) == 1 and ts == s:
    print ("accepted")
else:
    print ("wrong answer")
```

---
## [C. 小红的白色字符串](https://ac.nowcoder.com/acm/contest/76609/C)

思路: 状态机DP

引入三个状态

- 0, 末尾是空格，
- 1, 末尾是小写字母
- 2, 末尾是大写字母

具体转移和当前的字母的大小写性质有关

具体看代码

```python3 []
s = input()

from math import inf

dp0, dp1, dp2 = 0, inf, inf

n = len(s)
for c in s:
    if c >= 'A' and c <= 'Z':
        t0 = min(dp0, dp1, dp2) + 1
        t1 = inf
        t2 = dp0
        dp0, dp1, dp2 = t0, t1, t2
    else:
        t0 = min(dp0, dp1, dp2) + 1
        t1 = min(dp0, dp1, dp2)
        t2 = inf
        dp0, dp1, dp2 = t0, t1, t2

print (min(dp0, dp1, dp2))
```

---
## [D. 小红走矩阵](https://ac.nowcoder.com/acm/contest/76609/D)

思路: BFS

很板的一道题，只是在扩展下一步的时候，多了一个限制条件

```python3 []
h, w = list(map(int, input().split()))

maze = []
for _ in range(h):
    arr = list(input())
    maze.append(arr)
    
from math import inf
from collections import deque

vis = [[inf] * w for _ in range(h)]

deq = deque()
deq.append((0, 0))
vis[0][0] = 0

while len(deq) > 0:
    y, x = deq.popleft()
    for (dy, dx) in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        ty = y + dy
        tx = x + dx
        if 0 <= ty < h and 0 <= tx < w:
            if maze[y][x] != maze[ty][tx]:
                if vis[ty][tx] > vis[y][x] + 1:
                    vis[ty][tx] = vis[y][x] + 1
                    deq.append((ty, tx))

if vis[h - 1][w - 1] == inf:
    print (-1)
else:
    print (vis[h - 1][w - 1])
```

---
## [E. 小红的小红走矩阵](https://ac.nowcoder.com/acm/contest/76609/E)

思路: 构造一个弯

按照Case 1，构造一个”钩“，然后其余的保持和周边不同即可。

![alt](https://img-blog.csdnimg.cn/img_convert/5883271582eb08a76c94d9a009e0d566.png)


因为地图四色定律，所以只要”abcd“即可。


```python3 []
h, w = list(map(int, input().split()))

maze = [['A'] * w for _ in range(h)]
maze[0][0] = 'a'
maze[0][1] = 'b'
maze[0][2] = 'b'

maze[1][0] = 'a'
maze[1][1] = 'c'
maze[1][2] = 'c'

maze[2][0] = 'b'
maze[2][1] = 'c'

def compute(y, x):
    s = set()
    for (dy, dx) in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        ty, tx = y + dy, x + dx
        if 0 <= ty < h and 0 <= tx < w:
            if maze[ty][tx] != 'A':
                s.add(maze[ty][tx])
    for c in "abcd":
        if c not in s:
            return c
    return 'e'

for i in range(h):
    for j in range(w):
        if maze[i][j] == 'A':
            maze[i][j] = compute(i, j)

for i in range(h):
    print (''.join(maze[i]))
```

---

## [F. 小红的好子串询问](https://ac.nowcoder.com/acm/contest/76609/F)

思路: 经典结论 + 树状数组

不存在 大于等于2个子串的回文，

$$red的排列p，然后p循环出现$$

基于这个结论，对于某个查询[l, r], 则枚举red的排列，然后引入9个树状数组, 进行统计，求最小代价即可。


fenwicks[3][3]，表示位置余3，字母(按red=123）的9个树状数组


```python3 []
class BIT(object):
    def __init__(self, n):
        self.n = n
        self.arr = [0] * (n + 1)
    
    def query(self, p):
        r = 0
        while p > 0:
            r += self.arr[p]
            p -= p & -p
        return r
    
    def update(self, p, d):
        while p <= self.n:
            self.arr[p] += d
            p += p & -p

def toId(ch):
    if ch == 'r':
        return 0
    elif ch == 'e':
        return 1
    return 2


from math import inf
from itertools import permutations 

n, m = list(map(int, input().split()))
s = list(input())

trees = [ [ BIT(n) for _ in range(3) ] for _ in range(3)]

for i in range(n):
    trees[i % 3][toId(s[i])].update(i + 1, 1)

for _ in range(m):
    op, a1, a2 = input().split()
    if int(op) == 1:
        l, ch = int(a1) - 1, a2[0]
        trees[l % 3][toId(s[l])].update(l + 1, -1)
        s[l] = ch
        trees[l % 3][toId(s[l])].update(l + 1, 1)
    else:
        l, r = int(a1) - 1, int(a2) - 1
        
        res = inf
        for (t1, t2, t3) in list(permutations([0, 1, 2])):
            z1 = trees[(l + t1) % 3][0].query(r + 1) - trees[(l + t1) % 3][0].query(l)
            z2 = trees[(l + t2) % 3][1].query(r + 1) - trees[(l + t2) % 3][1].query(l)
            z3 = trees[(l + t3) % 3][2].query(r + 1) - trees[(l + t3) % 3][2].query(l)

            res = min(res, (r - l + 1) - (z1 + z2 + z3))
        print (res)
```

---
# 写在最后

![alt](https://img-blog.csdnimg.cn/img_convert/ca51c5a1e40f89936aac247773dfb74e.png)




