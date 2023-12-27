

[239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/description)

```python []
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        n = len(nums)
        if k > n:
            return []

        monostack = []
        res = []
        for i in range(0, n):
            while len(monostack) > 0 and monostack[0] <= i - k:
                monostack = monostack[1:]
            while len(monostack) > 0 and nums[monostack[-1]] <= nums[i]:
                monostack.pop()
            monostack.append(i)
            if i >= k - 1:
                res.append(nums[monostack[0]])

        return res
```

单调栈的应用




