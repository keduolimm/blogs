

[198. 打家劫舍](https://leetcode.cn/problems/house-robber/description)

思路：dp

线性状态机DP

$dp[i][s], s\in(0, 1)$

$$dp[i][0] = max(dp[i - 1][0], dp[i - 1][1])$$
$$dp[i][1] = dp[i - 1][0] + nums[i]$$

```python []
class Solution:
    def rob(self, nums: List[int]) -> int:
        # 线性dp
        n = len(nums)
        s0, s1 = 0, nums[0]
        for i in range(1, n):
            t0 = max(s0, s1)
            t1 = s0 + nums[i]
            s0, s1 = t0, t1
        return max(s0, s1)
```


一维dp也可以做的

$$dp[i] = max(dp[i - 1], dp[i - 2] + nums[i)$$


