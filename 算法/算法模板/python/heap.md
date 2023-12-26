

---
# python优先队列的使用

## 堆的初始化
```python  []
import heapq

list = [1, 3, 2]
heapq.heapify(list)
```


## 堆的push/pop
```python  []
import heapq

list = [1, 3, 2]
heapq.heapify(list)

# push
heapq.heappush(list, 6)
while len(list) > 0:
   # pop
   cur = heapq.heappop(list)
```

## 优先队列的排序

默认是最小堆，如果要改成最大堆只能添加'-'号

多维度的排序, 建议使用tuple

```python  []
import heapq

list = [(1, 2), (3, 4)]
heapq.heapify(list)

# push
heapq.heappush(list, (5, 6))
while len(list) > 0:
   # pop
   cur = heapq.heappop(list)
```


---
# 验证

[力扣 506 相对名次](https://leetcode.cn/problems/relative-ranks/description/)

```python []
class Solution:
    def findRelativeRanks(self, score: List[int]) -> List[str]:

        players = []
        for i in range(len(score)):
            players.append((-score[i], i))

        heapq.heapify(players)

        cnt = 1
        n = len(score)
        res = [""] * n
        while len(players) > 0:
            cur = heapq.heappop(players)
            if cnt == 1:
                res[cur[1]] = "Gold Medal"
            elif cnt == 2:
                res[cur[1]] = "Silver Medal"
            elif cnt == 3:
                res[cur[1]] = "Bronze Medal"
            else:
                res[cur[1]] = str(cnt)
            cnt += 1
        
        return res
```

---

# 题单


# 参考文章

[python3中的heapq模块（堆排序）使用](https://blog.csdn.net/flyingluohaipeng/article/details/129728356) 