

[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/description)

分层bfs的常规写法


---

Python写法

```python []

```

---

C++写法

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
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (root == NULL) {
            return res;
        }
        deque<TreeNode *> deq;
        deq.push_back(root);
        while (!deq.empty()) {
            vector<int> tmp;
            int sz = deq.size();
            for (int i = 0; i < sz; i++) {
                TreeNode *ptr = deq.front();
                deq.pop_front();
                tmp.push_back(ptr->val);

                if (ptr->left != NULL) deq.push_back(ptr->left);
                if (ptr->right != NULL) deq.push_back(ptr->right);
            }
            res.push_back(tmp);
        }
        return res;
    }
};
```

c++的deque

```c++
front()
back()

push_back()
pop_front()  // 返回值为null
```