
[56. 合并区间](https://leetcode.cn/problems/merge-intervals/description)

区间的合并

- 排序(偏序)
- 基于stack进行合并

```python []
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        intervals.sort(key = lambda x: x[0])

        stack = []
        for e in intervals:
            p, a = e[0], e[1]
            while len(stack) > 0 and stack[-1][1] >= p:
                cur = stack.pop() 
                p = min(p, cur[0])
                a = max(a, cur[1])
            stack.append([p, a])

        return stack
```