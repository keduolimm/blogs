


[101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/description)

```python []
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:

        def dfs(lroot: Optional[TreeNode], rroot: Optional[TreeNode]) -> bool:
            if lroot == None and rroot == None:
                return True
            elif lroot == None or rroot == None:
                return False
            
            if lroot.val != rroot.val:
                return False
            
            return dfs(lroot.left, rroot.right) and dfs(lroot.right, rroot.left)

        if root == None:
            return True
        return dfs(root.left, root.right)
```