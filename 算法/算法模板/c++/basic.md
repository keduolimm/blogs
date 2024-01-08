

--- 

设置浮点数的精度

```c++ []
cout << fixed << setprecision(10);
```

---
## 函数闭包

```c++ []
function<int(int)> dfs = [&](int p) {
    if (p <= 0) return 0;
    return p + dfs(p - 1);
};
```


---

## 自定义排序

```c++ []
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        sort(nums.begin(), nums.end(), [&](int a, int b) {
            return a < b;
        });
        return nums[nums.size() / 2];
    }
};```
