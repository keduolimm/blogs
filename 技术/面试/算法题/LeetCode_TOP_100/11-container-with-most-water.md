

[11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/description/)

```python []
class Solution:
    def maxArea(self, height: List[int]) -> int:
        i, j = 0, len(height) - 1
        res = min(height[i], height[j]) * (j - i)
        while i < j:
            if height[i] <= height[j]:
                i += 1
            else:
                j -= 1
            res = max(res, min(height[i], height[j]) * (j - i))
        return res            
```

后记，突然发现这题拿不下, T_T

这题如果使用单调栈的话，时间复杂度好像减不了

这个双指针真的太强了

$$min(height[i], height[j]) * (j - i)$$

然后决策有贪心的成分在




