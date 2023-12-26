

[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/description)

```python []
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        st = set(nums)

        res = 0
        for v in st:
            if v - 1 not in st:
                v2 = v + 1
                while v2 in st:
                    v2 += 1
                res = max(res, v2 - v)
        
        return res
```

借助hash, 其实实际上形成了区间，从左端点的特征入手

这样能构建一个O(n)的时间复杂度



