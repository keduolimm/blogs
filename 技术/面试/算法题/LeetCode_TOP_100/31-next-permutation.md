


[31. 下一个排列](https://leetcode.cn/problems/next-permutation/description)

找第一个邻近的逆序对

然后reverse

然后swap


```python  []
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """

        n = len(nums)
        pos = -1
        for i in range(n - 1, 0, -1):
            if nums[i - 1] < nums[i]:
                pos = i
                break
        
        if pos == -1:
            nums.reverse()
            return
        
        # 均摊为o(1)
        # 1. reverse
        s, e = pos, n - 1
        while s < e:
            nums[s], nums[e] = nums[e], nums[s]
            s += 1
            e -= 1
        # 2. swap
        pos2 = pos
        for i in range(pos, n):
            if nums[pos - 1] < nums[i]:
                pos2 = i
                break
        
        nums[pos - 1], nums[pos2] = nums[pos2], nums[pos - 1]
```