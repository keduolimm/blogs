# 第 376 场周赛 解题报告 | 珂学家 | 中位数定律场


# 前言

---
## 整体评价
这场是中位数定律场，如果有人不熟悉这个结论，那就容易翻车。T4其实在牛客做过，传智杯上也做过一次，^_^.

---
## [T1. 找出缺失和重复的数字](https://leetcode.cn/contest/weekly-contest-376/problems/find-missing-and-repeated-values/)

也有多种解法

- 空间换时间
构建一个全hash数组，然后计数

- 时间换空间
排序后+扫描

这是用满hash数组计数来实现
```java
class Solution {
    public int[] findMissingAndRepeatedValues(int[][] grid) {
        int n = grid.length;
        int[] hash = new int[n * n + 1];
        
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                hash[grid[i][j]]++;
            }
        }
        int a = -1, b = -1;
        for (int i = 1; i <= n * n; i++) {
            if (hash[i] == 2) a = i;
            else if (hash[i] == 0) b = i;
        }
        return new int[] {a, b};
    }
}
```

---

## [T2. 划分数组并满足最大差限制](https://leetcode.cn/contest/weekly-contest-376/problems/divide-array-into-arrays-with-max-difference/)

这题是子序列，不是子数组

所以万事不决，先排序

然后进行对比构造

```java
class Solution {
    public int[][] divideArray(int[] nums, int k) {
        int n = nums.length;
        Arrays.sort(nums);
        for (int i = 0; i < n; i += 3) {
            if (Math.abs(nums[i] - nums[i + 1]) > k 
                || Math.abs(nums[i] - nums[i + 2]) > k 
                || Math.abs(nums[i + 1] - nums[i + 2]) > k)  {
                return new int[0][];
            }
        }
        
        int[][] res = new int[n / 3][3];
        for (int i = 0; i < n / 3; i ++) {
            res[i][0] = nums[i * 3];
            res[i][1] = nums[i * 3 + 1];
            res[i][2] = nums[i * 3 + 2];
        }
        return res;
    }
}
```

---

## [T3. 使数组成为等数数组的最小代价](https://leetcode.cn/contest/weekly-contest-376/problems/minimum-cost-to-make-array-equalindromic/)

回文数，虽然在1e9的范围内，实际上并不多

这个可以从构造的角度去求出来

从[1, 10000]出发枚举，然后镜像构造

而对于目标解

$$|a_1 - x| + |a_2 - x| + ... + |a_n - x| 最小$$

本质上也是中位数定理，只不过这边中位数不见得是回文数

不过好的地方在于，这个数组的中位数，其目标解一定在回文数最接近的左右两数之间

```java []
class Solution {

    static TreeSet<Long> range = new TreeSet<>();
    static long limit = (long) 1e9;
    
    static {
        for (int i = 1; i < 10; i++) {
            range.add((long)i);
        }
        
        for (int i = 1; i <= 10000; i++) {
            String r = String.valueOf(i);
            String rv = new StringBuilder(r).reverse().toString();
            long lv = Long.valueOf(r + rv);
            if (lv < limit) {
                range.add(lv);
            }
            for (int j = 0; j < 10; j++) {
                char c = (char)('0' + j);
                String s = r + c + rv;
                long lv2 = Long.valueOf(s);
                if (lv2 < limit) {
                    range.add(lv2);
                }
            }
        }
    }

    long cost(Long v, int[] nums) {
        if (v == null) return Long.MAX_VALUE / 10;
        long res = 0;
        for (int num: nums) {
            res += Math.abs(num - v);
        }
        return res;
    }

    long evaluate(long v, int[] nums) {
        Long k1 = range.floor(v);
        Long k2 = range.ceiling(v);
        return Math.min(cost(k1, nums), cost(k2, nums));
    }

    public long minimumCost(int[] nums) {
        Arrays.sort(nums);
        int mid = nums.length / 2;
        if (nums.length % 2 == 1) {
            // 奇数个，一定是 n / 2
            return evaluate(nums[mid], nums);
        } else {
            // 偶数个，则为 n/2 - 1, n/2 从这两个点出发
            return Math.min(evaluate(nums[mid - 1], nums), evaluate(nums[mid], nums));
        }
    }
}
```

```python []
import bisect
from typing import List

arr = []
for i in range(1, 10 ** 5):
    s = str(i)
    s1 = s[:-1] + s[::-1]
    s2 = s + s[::-1]
    if int(s1) < 1e9:
        arr.append(int(s1))
    if int(s2) < 1e9:
        arr.append(int(s2))

arr.sort()


class Solution:
    def minimumCost(self, nums: List[int]) -> int:
        # 中位数定律
        nums.sort()
        n = len(nums)
        mid = nums[n // 2]

        def evaluate(x: int) -> int:
            tmp = 0
            for v in nums:
                tmp += abs(v - x)
            return tmp

        pos = bisect.bisect_left(arr, mid)
        
        if pos < len(arr) and arr[pos] == mid:
            return evaluate(mid)

        r1 = evaluate(arr[pos - 1]) if pos > 0 else inf
        r2 = evaluate(arr[pos]) if pos < len(arr) else inf
        return min(r1, r2)
```

---

## [T4. 执行操作使频率分数最大](https://leetcode.cn/contest/weekly-contest-376/problems/apply-operations-to-maximize-frequency-score/)

这题方法蛮多的

但是核心的原则就是

$$中位数定律$$

有如下几种解法

- 滑窗+中位数定律
  - 枚举左端点
  - 枚举中点
  - 枚举右端点
- 二分+中位数定律

这个解法是基于滑窗+中位数，枚举左端点来求解的

```java
class Solution {
    
    // 前置和
    long[] pre;
    
    // 中位数定律
    long evalute(int s, int e, int[] nums) {
        int m = s + (e - s) / 2;
        long r1 = (long)(m - s + 1) * nums[m] - (pre[m + 1] - pre[s]);
        long r2 = (pre[e + 1] - pre[m]) - (long)(e - m + 1) * nums[m];
        return r1 + r2;
    } 
    
    // 中位数定理+滑窗
    public int maxFrequencyScore(int[] nums, long k) {
        int n = nums.length;
        Arrays.sort(nums);
        
        pre = new long[n + 1];
        for (int i = 0; i < n; i++) {
            pre[i + 1] = pre[i] + nums[i];
        }
        
        int res = 0;
        int last = 0;
        for (int i = 0; i < n; i++) {
            while (last < n && evalute(i, last, nums) <= k) {
                last++;
            }
            
            res = Math.max(res, last - i);
        }
        
        return res;
    }
    
}
```

---

# 写在最后



