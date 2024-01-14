
# string

- find

子字符串的匹配

- push_back

用于字符串的拼接


# vector

- push_back

- pop_back

# map

- lower_bound 

返回 >= 的最小的那个迭代器

- upper_bound

返回 <= 的最大的那个迭代器


- 如何获取最小key

```c++
map<int, int> mp;
auto first = mp.begin();
```

- 如何获取最大key

```c++
map<int, int> mp;
auto last = --mp.end();
```


# set

- insert

- erase

- find

- lower_bound

- upper_bound


# multiset

- erase

如果指定值的话，是删除所有的相同的值

如果是迭代器的话，就删除其中一个

# deque

- push_back

- pop_front


# unordered_map

需要指定hash函数

定义仿函数
```c++ []

```

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

