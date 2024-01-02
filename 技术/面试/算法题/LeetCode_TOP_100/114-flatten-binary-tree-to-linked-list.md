

[114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/description)

c++解法

前序遍历，用栈实现非常的容易

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
    void flatten(TreeNode* root) {
        if (root == nullptr) return;
        stack<TreeNode *> stk;
        stk.push(root);

        TreeNode *pre = NULL;
        while (!stk.empty()) {
            TreeNode *cur = stk.top();
            stk.pop();

            if (cur->right != NULL) {
                stk.push(cur->right);
            }
            if (cur->left != NULL) {
                stk.push(cur->left);
            }

            cur->left = cur->right = NULL;
            if (pre != NULL) {
                pre->right = cur;
            }
            pre = cur;
        }

        root->left = NULL;
    }
};
```