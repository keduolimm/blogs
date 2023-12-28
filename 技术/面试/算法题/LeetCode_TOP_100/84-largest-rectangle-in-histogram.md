
[84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/description)

前后缀拆解

单调栈

```python []
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        # 单调栈

        n = len(heights)
        monostack1 = [] 
        left = [0] * n

        for i in range(n):
            h = heights[i]
            while len(monostack1) > 0 and monostack1[-1][1] >= h:
                monostack1.pop()
            
            left[i] = 0 if len(monostack1) == 0 else monostack1[-1][0] + 1
            monostack1.append((i, h))

        
        monostack2 = [] 
        right = [0] * n

        for i in range(n -1, -1, -1):
            h = heights[i]
            while len(monostack2) > 0 and monostack2[-1][1] >= h:
                monostack2.pop()
            
            right[i] = n - 1 if len(monostack2) == 0 else monostack2[-1][0] - 1
            monostack2.append((i, h))

        res = 0
        for i in range(n):
            span = right[i] - left[i] + 1
            res = max(res, span * heights[i])

        return res
```