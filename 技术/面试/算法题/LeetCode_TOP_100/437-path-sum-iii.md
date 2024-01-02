

[437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/description)

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
    int pathSum(TreeNode* root, int targetSum) {
        map<long long, int> hash;
        hash[0] = 1;
        return dfs(root, 0l, targetSum, hash);
    }

    int dfs(TreeNode *root, long long acc, int targetSum, map<long long, int> &hp) {
        int res = 0;
        if (root == nullptr) return 0;
        acc += root->val;
        res += hp[acc - targetSum];

        hp[acc]++;
        if (root->left != nullptr) {
            res += dfs(root->left, acc, targetSum, hp);
        }
        if (root->right != nullptr) {
            res += dfs(root->right, acc, targetSum, hp);
        }
        hp[acc]--;
        return res;
    }

};
```