


[1. 两数之和](https://leetcode.cn/problems/two-sum/description/)

```python []
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        n = len(nums)
        arr = [(nums[i], i) for i in range(n)]
        arr.sort(key=lambda x: x[0])
        i, j = 0, n - 1
        while i < j:
            if arr[i][0] + arr[j][0] == target:
                return [arr[i][1], arr[j][1]]
            elif arr[i][0] + arr[j][0] < target:
                i += 1
            else:
                j -= 1
        return []
```

时间复杂度为O(nlogn), 在排序上

当然hash是最方便的

