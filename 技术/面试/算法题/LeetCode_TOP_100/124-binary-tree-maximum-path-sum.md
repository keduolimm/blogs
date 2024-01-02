

[124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/description)



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
    int ans = -0x3f3f3f3f;
    int maxPathSum(TreeNode* root) {
        dfs(root);
        return ans;
    }

    int dfs(TreeNode *root) {
        if (root == nullptr) return 0;

        int r1 = 0;
        if (root->left != nullptr) {
            r1 = dfs(root->left);
        }
        int r2 = 0;
        if (root->right != nullptr) {
            r2 = dfs(root->right);
        }
        ans = max(ans, root->val);
        ans = max(ans, root->val + r1 + r2);

        return max(0, max(root->val + r1, root->val + r2));
    }

};
```