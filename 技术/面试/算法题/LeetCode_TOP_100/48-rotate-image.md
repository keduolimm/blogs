

[48. 旋转图像](https://leetcode.cn/problems/rotate-image/description)

从特殊到一般

```python []
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """

        # 对称性
        # i, j -> j, n - 1 - i -> n - 1 - i, n - 1 - j -> n - 1 - i, j
        n = len(matrix)
        for i in range(n // 2):
            for j in range((n + 1) // 2):
                matrix[i][j], matrix[j][n - 1 -  i], matrix[n - 1 - i][n - 1 - j], matrix[n - 1 - j][i] = \
                matrix[n - 1 - j][i], matrix[i][j], matrix[j][n - 1 -i], matrix[n - 1 - i][n - 1 - j]

```