

[98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/description)

---
C++ 解法

在线验证

```C++ []
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
    bool isValidBST(TreeNode* root) {
        int lv = 0, rv = 0;
        return isValidBST2(root, &lv, &rv);
    }
    
    bool isValidBST2(TreeNode* root, int *lv, int *rv) {
        if (root == NULL) return true;

        *lv = root->val;
        *rv = root->val;
        if (root->left != NULL) {
            int lv1 = 0, rv1 = 0;
            if (root->left->val >= root->val) return false;
            if (!isValidBST2(root->left, &lv1, &rv1)) return false;
            if (rv1 >= root->val) return false;

            *lv = lv1;
        }
        if (root->right != NULL) {
            int lv2 = 0, rv2 = 0;
            if (root->right->val <= root->val) return false;
            if (!isValidBST2(root->right, &lv2, &rv2)) return false;
            if (lv2 <= root->val) return false;

            *rv = rv2;
        }

        return true;
    }
};
```

- 转化为中序遍历 + 验证

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
    bool isValidBST(TreeNode* root) {
        vector<int> vec;
        dfs(vec, root);
        int n = vec.size();
        for (int i = 0; i < n - 1; i++) {
            if (vec[i] >= vec[i + 1]) return false;
        }
        return true;
    }

    // 中序遍历
    void dfs(vector<int> &res, TreeNode *root) {
        if (root == NULL) return;
        dfs(res, root->left);
        res.push_back(root->val);
        dfs(res, root->right);
    }
};
```