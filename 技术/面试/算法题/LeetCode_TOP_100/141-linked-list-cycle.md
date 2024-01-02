

[141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/description)

```c++ []
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {

        ListNode *slow = head;
        ListNode *fast = head;

        while (slow != nullptr && fast != nullptr) {
            slow = slow->next;
            if (fast->next == nullptr) return false;
            fast = fast->next->next;
            if (slow == fast && slow != nullptr) return true;
        }

        return false;
        
    }
};
```