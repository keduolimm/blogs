


[287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/description)

```python []
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        hp = defaultdict(int)
        for v in nums:
            hp[v] += 1
            if hp[v] > 1:
                return v
        return -1
```