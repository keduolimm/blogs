

[78. 子集](https://leetcode.cn/problems/subsets/description)

思路: 二进制枚举

```python []
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        # 二进制枚举
        res = []
        n = len(nums)
        for i in range(1 << n):
            tmp = []
            for j in range(n):
                if (i & (1 << j)) != 0:
                    tmp.append(nums[j])
            res.append(tmp)
        return res
```