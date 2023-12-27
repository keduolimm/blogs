

[41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/description)

hash的一个应用

```python []
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        cnt = Counter()
        for v in nums:
            cnt[v] += 1
        ptr = 1
        while cnt[ptr] >= 1:
            ptr += 1

        return ptr
```

