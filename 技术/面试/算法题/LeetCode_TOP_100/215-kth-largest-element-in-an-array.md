

[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/description)

这个quick select还是挺麻烦的

```python []
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:

        def quickselect(l: int, r: int, m: int) -> int:
            p = nums[l]
            if l == r:
                return nums[l]
            i, j = l, r
            while i <= j:
                while i <= j and nums[i] < p:
                    i += 1
                while i <= j and nums[j] > p:
                    j -= 1
                if i <= j:
                    nums[i], nums[j] = nums[j], nums[i]
                    i += 1
                    j -= 1
            
            if m < i:
                return quickselect(l, i - 1, m)
            else:
                return quickselect(i, r, m)

        return quickselect(0, len(nums) - 1, len(nums) - k)
```
