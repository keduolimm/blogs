

[169. 多数元素](https://leetcode.cn/problems/majority-element/description)

摩尔投票

```python []
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        # 摩尔投票
        nums.sort()
        return nums[len(nums) // 2]
```