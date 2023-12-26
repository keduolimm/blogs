
# 前言

并查集的应用还是蛮多的

----

# 一般模板

一般cf，abc, 牛客, 喜欢 index-1

```python
class Dsu(object):
    
    def __init__(self, n: int):
        self.n = n 
        self.arr = [0] * (n + 1) # index-1
        self.group = n
        
    def find(self, u: int) -> int:
        if self.arr[u] == 0:
            return u
        self.arr[u] = self.find(self.arr[u])
        return self.arr[u]
    
    def union(self, u :int, v: int):
        a = self.find(u)
        b = self.find(v)
        if a != b:
            self.arr[a] = b
            self.group -= 1
            
    def groupNum(self) -> int:
        return self.group
```

如果是力扣，喜欢index-0

```python
class Dsu(object):
    
    def __init__(self, n: int):
        self.n = n 
        self.arr = [-1] * n # index-0
        self.group = n
        
    def find(self, u: int) -> int:
        if self.arr[u] == -1:
            return u
        self.arr[u] = self.find(self.arr[u])
        return self.arr[u]
    
    def union(self, u :int, v: int):
        a = self.find(u)
        b = self.find(v)
        if a != b:
            self.arr[a] = b
            self.group -= 1
            
    def groupNum(self) -> int:
        return self.group
```

---
# 验证

牛客周赛Round 22 

[小红的图上删边](https://ac.nowcoder.com/acm/contest/70996/D)


```python
import heapq

class Dsu(object):
    
    def __init__(self, n: int):
        self.n = n 
        self.arr = [0] * (n + 1) # index-1
        self.group = n
        
    def find(self, u: int) -> int:
        if self.arr[u] == 0:
            return u
        self.arr[u] = self.find(self.arr[u])
        return self.arr[u]
    
    def union(self, u :int, v: int):
        a = self.find(u)
        b = self.find(v)
        if a != b:
            self.arr[a] = b
            self.group -= 1
            
    def groupNum(self) -> int:
        return self.group
            
            
n, m = list(map(int, input().split()))
arr = list(map(int, input().split())) 

# 逆向思维
cnt2 = [0] * (n + 1)
cnt5 = [0] * (n + 1)

for i in range(len(arr)):
    v = arr[i]
    while v % 2 == 0:
        v /= 2
        cnt2[i + 1] += 1
    while v % 5 == 0:
        v /= 5
        cnt5[i + 1] += 1

res = 0
edges = []
for i in range(m):
    u, v = list(map(int, input().split()))
    edges.append((min(cnt2[u] + cnt2[v], cnt5[u] + cnt5[v]), u, v))
    res += min(cnt2[u] + cnt2[v], cnt5[u] + cnt5[v])

dsu = Dsu(n)
heapq.heapify(edges)

while dsu.groupNum() > 1 and len(edges):
    row = heapq.heappop(edges)
    if dsu.find(row[1]) != dsu.find(row[2]):
        dsu.union(row[1], row[2])
        res -= row[0]

print (res)
```



