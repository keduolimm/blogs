

[54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/description)

模拟即可

引入阻塞数组的概念

```python []
class Solution:

    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        h, w = len(matrix), len(matrix[0])

        vis = [[False] * w for _ in range(h)]

        d = 0
        dirs = [[0, 1], [1, 0], [0, -1], [-1, 0]]

        y, x = 0, 0
        res = []
        while (y >= 0 and y < h and x >= 0 and x < w) and not vis[y][x]:
            res.append(matrix[y][x])
            vis[y][x] = True

            y2, x2 = y + dirs[d][0], x + dirs[d][1]

            if y2 < 0 or y2 >= h or x2 < 0 or x2 >= w or vis[y2][x2]:
                d = (d + 1) % 4
                y2, x2 = y + dirs[d][0], x + dirs[d][1]

            y, x = y2, x2

        return res
```