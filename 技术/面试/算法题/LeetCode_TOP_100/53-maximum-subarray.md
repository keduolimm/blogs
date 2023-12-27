

[53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/description)

- DP解
- 贪心解

```python []
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        res = -inf
        dp = 0
        for v in nums:
            # dp[i] = max(dp[i - 1] + v, v)
            dp = max(dp + v, v)
            res = max(res, dp)
        return res
```

