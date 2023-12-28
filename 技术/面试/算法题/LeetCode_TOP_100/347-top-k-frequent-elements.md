

[347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/description)

```python []
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        cnt = Counter()
        for v in nums:
            cnt[v] += 1

        arr = [(k, v) for k, v in cnt.items()]
        arr.sort(key=lambda x: -x[1])

        return [arr[i][0] for i in range(k)]
            
```