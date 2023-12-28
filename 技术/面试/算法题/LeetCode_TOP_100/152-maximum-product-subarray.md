

[152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/description)

思路: dp

dp[i][0]最大, dp[i][1]最小

dp[i][0] = max(dp[i - 1][0] * nums[i], dp[i - 1][1] * nums[i], nums[i])

dp[i][1] = min(dp[i - 1][0] * nums[i], dp[i - 1][1] * nums[i], nums[i])


```python []
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        # 维护一个最小，一个最大的
        res = inf

        n = len(nums)
        s0, s1 = nums[0], nums[0]
        res = min(s0, s1)
        for i in range(1, n):
            s0, s1 = max(s0 * nums[i], s1 * nums[i], nums[i]), min(s0 * nums[i], s1 * nums[i], nums[i])
            res = max(res, s0)

        return res
```

另外的思路: 贪心

按0切分子区间

arr = [x, y, .., z]

如果负数为偶数个，则全乘

如果负数为奇数个，则分类讨论



