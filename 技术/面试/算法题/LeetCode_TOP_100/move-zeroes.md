

[283. 移动零](https://leetcode.cn/problems/move-zeroes/description)

```python []
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        j = 0
        for i in range(len(nums)):
            if nums[i] != 0:
                if j < i:
                    nums[j], nums[i] = nums[i], nums[j]
                j += 1
```

如果可以额外用其他的数组，相对就比较简单了

算是双指针的好题


