

[226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/description)


```python []
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:

        if root == None:
            return
        
        lptr, rptr = root.left, root.right

        root.left = self.invertTree(rptr)
        root.right = self.invertTree(lptr)

        return root
```