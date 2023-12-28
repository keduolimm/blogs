
[70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/description)

线性DP

$$dp[i] = dp[i - 2] + dp[i - 1]$$

```python []
class Solution:
    def climbStairs(self, n: int) -> int:
        dp = [0] * (n + 1)
        dp[0] = 1
        for i in range(1, n + 1):
            dp[i] = dp[i] + dp[i - 1]
            if i >= 2:
                dp[i] = dp[i] + dp[i - 2]
        return dp[n]
```