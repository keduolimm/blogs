

[45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/description)

思路：dp

```python []
class Solution:
    def jump(self, nums: List[int]) -> int:
        # 先看值域
        n = len(nums)
        dp = [inf] * n
        dp[0] = 0
        for i in range(n):
            for j in range(1, nums[i] + 1):
                if i + j >= n:
                    break
                dp[i + j] = min(dp[i + j], dp[i] + 1)
        return dp[n - 1]
```