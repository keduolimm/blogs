

[55. 跳跃游戏](https://leetcode.cn/problems/jump-game/description)

思路: 贪心

```python []
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        jump = nums[0]
        n = len(nums)
        for i in range(n):
            if i <= jump:
                jump = max(jump, i + nums[i])
        return jump >= n - 1
```