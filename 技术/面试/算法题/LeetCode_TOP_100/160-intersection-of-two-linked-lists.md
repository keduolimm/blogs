

[160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/description)


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
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        int n1 = lenLink(headA), n2 = lenLink(headB);
        while (n1 > n2) {
            headA = headA->next;
            n1--;
        }
        while (n1 < n2) {
            headB = headB->next;
            n2--;
        }
        while (headA != nullptr) {
            if (headA == headB) return headA;
            headA = headA->next;
            headB = headB->next;
        }
        return nullptr;
    }

    int lenLink(ListNode *head) {
        int cnt = 0;
        while (head != nullptr) {
            cnt++;
            head = head->next;
        } 
        return cnt;
    }

};
```