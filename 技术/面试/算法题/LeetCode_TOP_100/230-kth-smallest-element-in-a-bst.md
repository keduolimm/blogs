


[230. 二叉搜索树中第K小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/description)


----

C++解法

中序遍历(二叉搜索树)，这题只能常数级优化

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
    int kthSmallest(TreeNode* root, int k) {
        vector<int> vec;
        inorder(vec, root);
        if (k > vec.size()) return -1;
        return vec[k - 1];
    }

    void inorder(vector<int> &vec, TreeNode *root) {
        if (root == NULL) return;
        inorder(vec, root->left);
        vec.push_back(root->val);
        inorder(vec, root->right);
    }

};
```

常数级优化

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

private:
    int cnt = 0;
    int ans = 0;
    int k;

public:
    int kthSmallest(TreeNode* root, int k) {
        this->k = k;
        inorder(root);
        return ans;
    }

private:
    bool inorder(TreeNode *root) {
        if (root == NULL) return false;
        if (inorder(root->left)) return true;
        if (++cnt == k) {
            ans = root->val;
            return true;
        }
        if (inorder(root->right)) return true;
        return false;
    }

};
```

正解是，利用stack来模拟实现二叉树的中序遍历

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
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode *> stk;
        int cnt = 0;
        while (root != NULL || !stk.empty()) {
            while (root != NULL) {
                stk.push(root);
                root = root->left;
            }
            root = stk.top();
            stk.pop();
            if (++cnt == k) return root->val;
            root = root->right;
        }
        return -1;
    }
};
```