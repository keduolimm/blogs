
[73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/description)

如果把空间复杂度降低，需要引入一个特别的数字

这个数字，在原矩阵中，不存在，即可

这边引入row/col数组

```python []
class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """

        h, w = len(matrix), len(matrix[0])
        row = [False] * h
        col = [False] * w
        for i in range(h):
            for j in range(w):
                if matrix[i][j] == 0:
                    row[i] = True
                    col[j] = True

        for i in range(h):
            if row[i]:
                for j in range(w):
                    matrix[i][j] = 0
        for j in range(w):
            if col[j]:
                for i in range(h):
                    matrix[i][j] = 0
                
```