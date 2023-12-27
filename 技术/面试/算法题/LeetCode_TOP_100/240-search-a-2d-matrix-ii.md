

[240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/description)

```python []
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        h, w = len(matrix), len(matrix[0])
        y, x = 0, w - 1

        while y >= 0 and y < h and x >= 0 and x < w:
            if matrix[y][x] == target:
                return True
            elif matrix[y][x] > target:
                x -= 1
            else:
                y += 1
        return False
        
```