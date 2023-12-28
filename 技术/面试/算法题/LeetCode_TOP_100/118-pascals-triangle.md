

[118. 杨辉三角](https://leetcode.cn/problems/pascals-triangle/description)

简单dp

组合数计算

```python []
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        res = [[0] * i for i in range(1, numRows + 1)]
        for i in range(numRows):
            res[i][0] = res[i][i] = 1
            for j in range(1, i):
                res[i][j] = res[i - 1][j - 1] + res[i - 1][j]
        return res
```