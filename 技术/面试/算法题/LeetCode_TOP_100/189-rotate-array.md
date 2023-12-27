

[189. 轮转数组](https://leetcode.cn/problems/rotate-array/description)

- 裴蜀定律
循环节, 可以把空间优化到O(1)


```python []
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        if k % n == 0:
            return
        k = k % n

        # 裴蜀定律
        gv = gcd(k, n)

        for i in range(gv):
            s = i
            cur = nums[i]
            while (s + k) % n != i:
                ns = (s + k) % n
                tmp = nums[ns]
                nums[ns] = cur
                cur = tmp
                s = ns
            nums[i] = cur
```