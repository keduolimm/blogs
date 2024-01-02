

[206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/description)

prev, head, next

三者之间的关系，不需要借助dummpy模式

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
    ListNode* reverseList(ListNode* head) {
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