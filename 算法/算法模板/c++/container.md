
# string

find

# vector

push_back

pop_back

# map


# set

- insert

- erase

- find

- lower_bound

- upper_bound


# multiset


# deque


# unordered_map


# priority_queue

默认为最小堆, greater, 最大堆less

- top

- push

- pop

- size

- empty

```c++ []
class KthLargest {
public:
    priority_queue<int, vector<int>, greater<int> > pq;
    int k;

    KthLargest(int k, vector<int>& nums) {
        this->k = k;
        for (int v: nums) {
            if (pq.size() < k) {
                pq.push(v);
            } else if (pq.top() < v) {
                pq.pop();
                pq.push(v);
            }
        }
    }
    
    int add(int val) {
        if (pq.size() < k) {
            pq.push(val);
        } else if (pq.size() == k && pq.top() < val) {
            pq.pop();
            pq.push(val);
        }
        return pq.top();
    }
};

/**
 * Your KthLargest object will be instantiated and called as such:
 * KthLargest* obj = new KthLargest(k, nums);
 * int param_1 = obj->add(val);
 */
```

---

## array

array<int64, 4>

