

[105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description)

```c++ []
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return dfs(preorder, 0, preorder.size() - 1, inorder, 0, inorder.size());
    }


    TreeNode *dfs(vector<int>& preorder, int s1, int e1, vector<int>& inorder, int s2, int e2) {
        if (s1 > e1) return nullptr;
        if (s1 == e1) {
            return new TreeNode(preorder[s1]);
        }
        int head = preorder[s1];
        TreeNode *root = new TreeNode(head);
        int pos = 0;
        for (int i = s2; i <= e2; i++) {
            if (inorder[i] == head) {
                pos = i;
                break;
            }
        }

        int span = pos - s2;
        root->left = dfs(preorder, s1 + 1, s1 + span, inorder, s2, pos - 1);
        root->right = dfs(preorder, s1 + span + 1, e1, inorder, pos + 1, e2);

        return root;
    }

};
```