

[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/description)


```python []
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        
        res = 0

        def dfs(root: Optional[TreeNode]) -> int:
            nonlocal res
            if root == None:
                return 0
            v1 = dfs(root.left)
            v2 = dfs(root.right)

            res = max(res, v1 + v2)
            return max(v1, v2) + 1

        dfs(root)
        return res

        
```