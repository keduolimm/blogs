

[236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/description)


```c++ []
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        stack<TreeNode *> stk1;
        vector<TreeNode *> vec1;
        search(root, p, stk1, vec1);

        stack<TreeNode *> stk2;
        vector<TreeNode *> vec2;
        search(root, q, stk2, vec2);

        int cm = min(vec1.size(), vec2.size());
        for (int i = cm - 1; i >= 0; i--) {
            if (vec1[i] == vec2[i]) return vec1[i];
        }
        return NULL;
    }

    bool search(TreeNode *root, TreeNode *p, stack<TreeNode *> &stk, vector<TreeNode *> &vec) {
        stk.push(root);
        if (root == p) {
            while (!stk.empty()) {
                vec.push_back(stk.top());
                stk.pop();
            }
            reverse(vec.begin(), vec.end());
            return true;
        }
        if (root->left != NULL) {
            if (search(root->left, p, stk, vec)) return true;
        }
        if (root->right != NULL) {
            if (search(root->right, p, stk, vec)) return true;
        }
        stk.pop();
        return false;
    }

};
```