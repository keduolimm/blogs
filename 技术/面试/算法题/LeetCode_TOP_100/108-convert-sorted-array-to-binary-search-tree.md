


[108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/description)

---

---

C++代码

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
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        if (nums.size() == 0) {
            return NULL;
        }

        int n = nums.size();
        return travel(nums, 0, n - 1);
    }

    TreeNode *travel(vector<int> &nums, int l, int r) {
        if (l > r) return NULL;
        int m = l + (r - l) / 2;
        TreeNode *cur = new TreeNode(nums[m]);

        cur->left = travel(nums, l, m - 1);
        cur->right = travel(nums, m + 1, r);

        return cur;
    }

};
```