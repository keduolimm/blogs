

[300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/description)

简单dp

也可以LIS

```python []
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)
        dp = [0] * n
        res = 0
        for i in range(n):
            dp[i] = 1
            for j in range(i):
                if nums[i] > nums[j]:
                    dp[i] = max(dp[i], dp[j] + 1)
            res = max(res, dp[i])
        return res
```