
# 牛客周赛 Round 34 解题报告 | 珂学家 | 构造思维 + 置换环

----
# 前言

---

# 整体评价

好绝望的牛客周赛，彻底暴露了CF菜菜的本质，F题没思路，G题用置换环骗了50%, 这大概是唯一的亮点了。

---

## [A. 小红的字符串生成](https://ac.nowcoder.com/acm/contest/75766/A)

思路: 枚举

a,b两字符在相等情况下比较特殊

```python3 []
a, b = input().split()
if a == b:
    print (2)
    print (a)
    print (a + a)
else:
    print (4)
    print (a)
    print (b)
    print (a + b)
    print (b + a)
```
---

## [B. 小红的非排列构造](https://ac.nowcoder.com/acm/contest/75766/B)

思路: 脑筋急转弯

如果合法，就是0

如果不合法，就把第1项改为n+1（排列外的数）

```python3 []
n = int(input())
arr = list(map(int, input().split()))

si = set()
for v in arr:
    if 1 <= v <= n:
        si.add(v)
if len(si) != n:
    print (0)
else:
    print (1)
    print (1, n + 1)
```

---

## [C. 小红的数字拆解](https://ac.nowcoder.com/acm/contest/75766/C)

思路: 贪心

从高位到低位贪心即可

```python3 []
s = input()

arr = []

i = 0
while i < len(s):
    j = i
    while j < len(s):
        p = ord(s[j]) - ord('0')
        if p % 2 == 0:
            break
        j += 1
    arr.append(s[i:j+1])
    i = j + 1

arr.sort(key=lambda x: (len(x), x))

print (*arr, sep='\n')
```

---

## [D. 小红的陡峭值](https://ac.nowcoder.com/acm/contest/75766/D)

思路: 构造

这题是限制性很强的题， 绝对差值总和为1

对于不满足条件数组，可以立马返回 -1.

难点在于

$$如何构造合法数组$$

假定0元素(左右两侧都存在非0值)，和左侧元素保持一致（反证法使得该假设一定成立）

那就剩下一侧有非0值的0值，如何讨论呢？

- 如果绝对差值总和为1
    - 那就和左侧/右侧的那个非0值，保持一致
- 如果绝对差值总和为0
    - 那就选一边构造一个比左侧/右侧刚好大1的数


```python3 []
n = int(input())
arr = list(map(int, input().split()))

if arr == [0 for _ in range(n)]:
    if len(arr) == 1:
        print (-1)
    else:
        print (*([1] + [2] * (n - 1)), sep=' ')
else:
    # 计算当前的差值综合
    brr = [v for v in arr if v > 0]
    diff = 0
    for i in range(len(brr) - 1):
        diff += abs(brr[i] - brr[i+1])

    if diff > 1:
        print (-1)
    else:
        first, last = -1, -1
        for i in range(n):
            if arr[i] != 0:
                if first == -1:
                    first = i
                last = i 

        # 填充中间的值
        pre = arr[first]
        for i in range(first + 1, last):
            if arr[i] == 0:
                arr[i] = pre
            else:
                pre = arr[i]

        # 填充两端的值
        if diff == 1:
            # 需要两端保持绝对值差值为0
            for i in range(first):
                arr[i] = arr[first]
            for i in range(last + 1, n):
                arr[i] = arr[last]
            print (*arr, sep=' ')
        else:
            # 需要两端构建出一个绝对值差值1
            if first > 0:
                for i in range(first):
                    arr[i] = arr[first] + 1
                for i in range(last + 1, n):
                    arr[i] = arr[last]
                print (*arr, sep=' ')
            elif last + 1 < n:
                for i in range(last + 1, n):
                    arr[i] = arr[last] + 1
                print (*arr, sep=' ')
            else:
                print (-1)
```

---
## [E. 小红的树形 dp](https://ac.nowcoder.com/acm/contest/75766/E)

思路: BFS + 验证

属于诈骗题，因为题目一直再诱导你往树形DP上去靠

其实从bfs出发，进行交叉染色

然后对边上两点进行验证，如果存在同色，即不合法

```python3 []
n = int(input())
s = input()
ss = list(s)

g = [[] for _ in range(n)]

for _ in range(n - 1):
    u, v = list(map(int, input().split()))
    u -= 1
    v -= 1
    g[u].append(v)
    g[v].append(u)

from collections import deque

deq = deque()
for i in range(n):
    if ss[i] != '?':
        deq.append((i, ss[i]))
        
# 需要补充这种特殊情况
if len(deq) == 0:
    ss[0] = 'd'
    deq.append((0, ss[0]))

# BFS流程
while len(deq) > 0:
    idx, ch = deq.popleft()
    for v in g[idx]:
        if ss[v] == '?':
            ss[v] = 'd' if ch == 'p' else 'p'
            deq.append((v, ss[v]))

# 验证逻辑
ok = True
for i in range(n):
    ch = ss[i]
    if ch == '?':
        ok = False
        break
    for v in g[i]:
        if ch == 'd' and ss[v] != 'p':
            ok = False
            break
        if ch == 'p' and ss[v] != 'd':
            ok = False
            break

if not ok:
    print (-1)
else:
    print (''.join(ss))
```

---

## [F. 小红的矩阵构造](https://ac.nowcoder.com/acm/contest/75766/F)

思路: 异或特性

贴一下[皮神](https://ac.nowcoder.com/acm/contest/profile/786945593)的代码

代码即是说服力

```python3 []
n, m, x = map(int, input().split())
 
res = [[0] * m for _ in range(n)]
 
if x % 4 == 0:
    res[0][0] = res[0][1] = res[1][0] = res[1][1] = x // 4
    for ls in res:
        print(*ls)
elif x == 2:
    print(-1)
else:
    res[2][0] = res[2][1] = 1
    res[1][0] = res[1][2] = 1
    res[0][1] = res[0][2] = 1
    for i in [0, -1]:
        for j in [0, -1]:
            res[i][j] += (x - 6) // 4
    for ls in res:
        print(*ls)
```
---

## [G. 小红的元素交换](https://ac.nowcoder.com/acm/contest/75766/G)

思路: 置换环 + 找规律

假如这题没有交换颜色的限制，那这题就是裸的置换环

但是实际上，这题的核心框架依旧是

$$置换环$$

具体情况需要分类讨论

- 同一置换环(a个元素)中存在两种颜色, 则交换一定为a-1

- 同一置换环(a个元素)中只存在1种颜色, 则需要借助外部的非同色，这样为a-1+2=a+1


但是这样做，只能对大致50%+

为啥呢？

因为对于纯色R的置换环，和纯色W的置换环，其实可以互相借调，因此这一组合，可以减少2次不必要的交换。

因此在原先的基础上，需要优化减掉

$$min(纯色R的群数，纯色W的群数) * 2$$

```python3 []
n = int(input())
arr = list(map(int, input().split()))
s = input()

from collections import Counter

if [i + 1 for i in range(n)] != arr and len(Counter(s)) == 1:
    print(-1)
else:
    res = 0
    white, red = 0, 0
    for i in range(n):
        if arr[i] == i + 1:
            continue
        else:
            # 置换环核心逻辑
            state = 0
            tmp = 0
            while arr[i] != i + 1:
                p1, p2 = i, arr[i] - 1
                arr[p1], arr[p2] = arr[p2], arr[p1]
                tmp += 1
                state |= 2 if s[p1] == 'R' else 1
                state |= 2 if s[p2] == 'R' else 1
                
            # 分类讨论置换环的纯色情况
            if state == 3:
                res += tmp
            else:
                # 纯色，需要借调外部力量
                res += tmp + 2
                if state == 1:
                    white += 1
                else:
                    red += 1

    print(res - min(white, red) * 2)
```

---

# 写在最后
![alt](https://uploadfiles.nowcoder.com/images/20240225/446702330_1708870306595/D2B5CA33BD970F64A6301FA75AE2EB22)

