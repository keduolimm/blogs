
# 前言

python的递归栈深度很浅，只有1000

```python 
sys.getrecursionlimit()
```

尽管可以调大，但是很多oj，还是不行

```python 
sys.setrecursionlimit(10000)
```

----
# 解法

装饰器

```python []
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
```

使用的话也很简单，在dfs函数上，注解该装饰器即可

同时也要改造dfs函数，添加yield语义

---

# 验证

[牛客周赛25 round D 小红树](https://ac.nowcoder.com/acm/contest/72266/D)

```python []
from types import GeneratorType

# 递归栈优化
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


n = int(input())
s = " " + input()

g = [[] for _ in range(n + 1)]

for i in range(n - 1):
    u, v = list(map(int, input().split()))
    g[u].append(v)
    g[v].append(u)
    

dp = [0] * (n + 1)
depth = [0] * (n + 1)

@bootstrap
def dfs(u: int, fa: int, d: int):
    depth[u] = d
    res = 1
    for v in g[u]:
        if v == fa:
            continue
        yield dfs(v, u, d + 1)
        if s[u] == s[v]:
            res += dp[v] - 1
        else:
            res += dp[v]
    dp[u] = res
    yield
    
dfs(1, -1, 1)

ans = 0
for i in range(1, n + 1):
    for v in g[i]:
        if depth[v] > depth[i]:
            if s[i] == s[v]:
                ans += abs(dp[1] - dp[v] + 1 - dp[v])
            else:
                ans += abs(dp[1] - dp[v] - dp[v])
                
print (ans)
```

---

# 题单








