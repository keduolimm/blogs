
[136. 只出现一次的数字](https://leetcode.cn/problems/single-number/description)

异或知识点

```python []
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        res = 0
        for v in nums:
            res ^= v
        return res
```

```python
reduce(lambda a, b: a ^ b, nums)
```