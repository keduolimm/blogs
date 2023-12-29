

[75. 颜色分类](https://leetcode.cn/problems/sort-colors/description)

```python []
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """

        n = len(nums)
        i, j = 0, n - 1
        while i < j:
            while i < j and nums[i] == 0:
                i += 1
            while i < j and nums[j] != 0:
                j -= 1
            if i < j:
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
                j -= 1
        
        i, j = 0, n - 1
        while i < j:
            while i < j and nums[i] != 2:
                i += 1
            while i < j and nums[j] == 2:
                j -= 1
            if i < j:
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
                j -= 1

        
```