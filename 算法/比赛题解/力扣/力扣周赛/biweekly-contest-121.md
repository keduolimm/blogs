

# 第 121 场双周赛 解题报告 | 珂学家 | 数位DP

---

# 前言

--- 

# 整体评价

T3, T4 都是典题

---

## [T1. 大于等于顺序前缀和的最小缺失整数](https://leetcode.cn/contest/biweekly-contest-121/problems/smallest-missing-integer-greater-than-sequential-prefix-sum/)

思路: 模拟

```c++ []
class Solution {
public:
    int missingInteger(vector<int>& nums) {
        set<int> s(nums.begin(), nums.end());
        int acc = nums[0];
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i - 1] + 1 != nums[i]) break;
            acc += nums[i];
        }
        
        while (s.find(acc) != s.end()) {
            acc ++;
        }
        return acc;
    }
};
```

---

## [T2. 使数组异或和等于 K 的最少操作次数](https://leetcode.cn/contest/biweekly-contest-121/problems/minimum-number-of-operations-to-make-array-xor-equal-to-k/)

思路: 思维题

设 $k ^ \prod_{i=0}^{i=n-1} nums[i]$ 的1个数

```c++ []
class Solution {
public:
    int minOperations(vector<int>& nums, int k) {
        int r = 0;
        for (int v: nums) r ^= v;
        
        r = r ^ k;
        
        int cnt = 0;
        while (r > 0) {
            r = r & (r - 1);
            cnt++;
        }
        return cnt;
    }
};
```

---

## [T3. 使 X 和 Y 相等的最少操作次数](https://leetcode.cn/contest/biweekly-contest-121/problems/minimum-number-of-operations-to-make-x-and-y-equal/)

思路: BFS

BFS的状态数10^6内

如果x,y值域扩大的话，那就很变难了，需要用dfs来求解

```c++ []
class Solution {
public:
    int minimumOperationsToMakeEqual(int x, int y) {
        
        if (x <= y) return y - x;
        
        int inf = 0x3f3f3f3f;
        vector<int> vec(x + 11 + 1, inf);
        
        vec[x] = 0;
        deque<int> deq;
        deq.push_back(x);
        
        while (!deq.empty()) {
            int c = deq.front();
            deq.pop_front();
            if (c - 1 >= 0 && vec[c - 1] > vec[c] + 1) {
                vec[c - 1] = vec[c] + 1;
                deq.push_back(c - 1);
            } 
            if (c + 1 <= x + 11 && vec[c + 1] > vec[c] + 1) {
                vec[c + 1] = vec[c] + 1;
                deq.push_back(c + 1);
            }
            if (c % 5 == 0 && vec[c / 5] > vec[c] + 1) {
                vec[c / 5] = vec[c] + 1;
                deq.push_back(c / 5);
            }
            if (c % 11 == 0 && vec[c / 11] > vec[c] + 1) {
                vec[c / 11] = vec[c] + 1;
                deq.push_back(c / 11);
            }
        }
        
        return vec[y];
    }
};
```


--- 

## [T4. 统计强大整数的数目](https://leetcode.cn/contest/biweekly-contest-121/problems/count-the-number-of-powerful-integers/)

思路: 数位DP

```c++ []
using int64 = long long;

class Solution {
public:
    long long numberOfPowerfulInt(long long start, long long finish, int limit, string s) {
        return solve(finish, limit, s) - solve(start - 1, limit, s);
    }

    int64 solve(int64 v, int limit, string s) {

        vector<int> arr;
        for (int64 x = v; x > 0; x /= 10) arr.push_back((int)(x % 10));
        reverse(arr.begin(), arr.end());

        int n = arr.size(), m = s.length();
        if (n < s.length()) return 0;
        if (n == m) {
            int sv = atoi(s.c_str());
            return sv <= v ? 1LL : 0LL;
        }        
        
        vector<int64> dp(n, -1);
        function<int64(int, bool, bool)> dfs = [&](int idx, bool isNum, bool isLimit) {
            if (idx + m == n) {
                if (isLimit) {
                    for (int i = idx; i < n; i++) {
                        if (arr[i] < s[i - idx] - '0') return 0LL;
                        else if (arr[i] > s[i - idx] - '0') return 1LL;
                    }
                }
                return 1LL;
            }
            int64 res = 0;
            if (!isNum) {
                res += dfs(idx + 1, false, false);
            }
            if (isNum && !isLimit) {
                if (dp[idx] != -1) return dp[idx];
            }

            int s = isNum ? 0 : 1;
            int e = isLimit ? min(arr[idx], limit) : limit;
            for (int i = s; i <= e; i++) {
                res += dfs(idx + 1, true, isLimit && i == arr[idx]);
                
            }

            if (isNum && !isLimit) {
                dp[idx] = res;
            }
            return res;
        };


        int64 res = dfs(0, false, true);
         
        return res;
    }

};
```

# 写在最后

