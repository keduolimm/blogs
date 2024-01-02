

[234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/description)

```c++ []
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    // 这个函数有副作用，会改变head原先的列表
    bool isPalindrome(ListNode* head) {
        vector<int> vec1 = travel(head);
        vector<int> vec2 = travel(reverse(head));
        return vec1 == vec2;
    }

    vector<int> travel(ListNode *head) {
        vector<int> res;
        while (head != nullptr) {
            res.push_back(head->val);
            head = head->next;
        }   
        return res;
    }

    ListNode *reverse(ListNode *head) {
        ListNode *prev = nullptr;
        while (head != nullptr) {
            ListNode *tmp = head->next;
            head->next = prev;
            prev = head;
            head = tmp;
        }
        return prev;
    }

};
```


