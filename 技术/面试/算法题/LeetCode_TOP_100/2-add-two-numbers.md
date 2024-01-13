
[2. 两数相加](https://leetcode.cn/problems/add-two-numbers/description)


- c++ 代码
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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *dummy = new ListNode();
        int c = 0;
        ListNode *prev = dummy;
        while (l1 != nullptr && l2 != nullptr) {
            int t = (l1->val + l2->val + c);
            int d = t % 10;
            c = t / 10;

            prev->next = new ListNode(d);
            prev = prev->next;

            l1 = l1->next;
            l2 = l2->next;
        }
        while (l1 != nullptr) {
            int t = (l1->val + c);
            int d = t % 10;
            c = t / 10;

            prev->next = new ListNode(d);
            prev = prev->next;

            l1 = l1->next;
        }

        while (l2 != nullptr) {
            int t = (l2->val + c);
            int d = t % 10;
            c = t / 10;

            prev->next = new ListNode(d);
            prev = prev->next;

            l2 = l2->next;
        }
        
        if (c > 0) {
            prev->next = new ListNode(c);
            prev = prev->next; // 
        }
        return dummy->next;
    }
};
```

---

- python

```python[] 

```

