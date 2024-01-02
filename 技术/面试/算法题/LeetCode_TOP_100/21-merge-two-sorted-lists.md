

[21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/description)

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
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode *dummy = new ListNode();
        ListNode *prev = dummy;
        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val <= list2->val) {
                ListNode *tmp = list1->next;
                prev->next = list1;
                list1->next = nullptr;
                prev = list1;
                list1 = tmp;
            } else {
                ListNode *tmp = list2->next;
                prev->next = list2;
                list2->next = nullptr;
                prev = list2;
                list2 = tmp;   
            }
        }
        if (list1 != nullptr) {
            prev->next = list1;
        }
        if (list2 != nullptr) {
            prev->next = list2;
        }
        return dummy->next;
    }
};
```