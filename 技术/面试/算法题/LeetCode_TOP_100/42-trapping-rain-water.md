

[42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/description)

```python []
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        suf = [0] * n
        suf[n - 1] = height[n - 1]
        for i in range(n - 2, -1, -1):
            suf[i] = max(suf[i + 1], height[i])
        
        res = 0
        pre = 0
        for i in range(n):
            h = min(pre, suf[i])
            if h > height[i]:
                res += h - height[i]
            pre = max(pre, height[i])

        return res
```

前后缀拆解

本质就是，每个slot，找到左侧/右侧最大值，觉得它的水高度
