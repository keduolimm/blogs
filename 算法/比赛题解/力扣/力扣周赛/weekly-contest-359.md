# 第 359 场周赛 解题报告 | 珂学家 | 线性DP + 分组&双指针


---

# 前言

![image.png](https://pic.leetcode.cn/1692503355-MTWXJb-image.png)


人工智能究竟能不能拥有和人一样的“爱”。

看完这本书的我觉得，这种爱，人工智能不应该去渴求拥有。

---

# 整体评价

其实还是T2有趣，凭感觉猜了一个结论。T3是一个线性DP, 可以从值域的角度切入，如果值域范围很大(可以离散化)，当然这边有个小技巧。T4分组就行，然后组内双指针滑窗即可。

---

## [T1. 判别首字母缩略词](https://leetcode.cn/contest/weekly-contest-359/problems/check-if-a-string-is-an-acronym-of-words/)

签到题，或者说API题

```java
class Solution {
    public boolean isAcronym(List<String> words, String s) {
        return words.stream().map(x -> x.substring(0, 1)).collect(Collectors.joining("")).equals(s);
    }
}
```

---

## [T2. k-avoiding 数组的最小总和](https://leetcode.cn/contest/weekly-contest-359/problems/determine-the-minimum-sum-of-a-k-avoiding-array/)

盲猜一个结论
> 满足的序列为 [1,...,k/2], [k,...,]分段序列

- 如果$n \le k/2$, 则为 $\sum_{i=1}^{i=n} i$, 即$n*(n+1)/2$
- 如果$n \gt k/2$, 则为2部分的总和

```java
class Solution {
    
    // [a,...b]区间和
    int accumulate(int a, int b) {
        return (a + b) * (b - a + 1) / 2;
    }
    
    public int minimumSum(int n, int k) {
        int k2 = k / 2;
        if (n <= k2) {
            return accumulate(1, n);
        } else {
            int r1 = accumulate(1, k2);
            int r2 = accumulate(k, k + (n - k2 - 1));
            return r1 + r2;
        }
    }
    
}
```

---
## [T3. 销售利润最大化](https://leetcode.cn/contest/weekly-contest-359/problems/maximize-the-profit-as-the-salesman/)

经典的线性DP题

设计状态 $opt[i]$, $i表示前i个房屋最大的收益$

那么状态转移为

> $opt[i] = max(opt[i - 1], opt[i - offer[j][0] - 1] + offer[j][2]),  offer[j][1] == i$

时间复杂度为 $O(n+m+mlog(m))$, n为值域房间数, m为买家数

```java
class Solution {
    
    public int maximizeTheProfit(int n, List<List<Integer>> offers) {

        // 按右端点进行排序
        Collections.sort(offers, Comparator.comparingInt(x -> x.get(1)));
        
        // 这边主动偏移一位
        int[] dp = new int[n + 1];
        
        int j = 0;
        for (int i = 1; i <= n; i++) {
            dp[i] = dp[i - 1];
            
            // 双指针
            while (j < offers.size() && offers.get(j).get(1) + 1 == i) {
                int y = offers.get(j).get(1) + 1;
                int x = offers.get(j).get(0);
                
                dp[y] = Math.max(dp[y], dp[x] + offers.get(j).get(2));
                j++;
            }
            
        }
        
        return dp[n];
    
    }
    
}
```


---
## [T4. 找出最长等值子数组](https://leetcode.cn/contest/weekly-contest-359/problems/find-the-longest-equal-subarray/)

这个分组就行，然后组内进行枚举左右两个端点。

那枚举左右两个端点，其实是可以用双指针进行优化，所以这题思路其实非常的直接。

```java
class Solution {
    
    public int longestEqualSubarray(List<Integer> nums, int k) {

        // 先进行分组
        Map<Integer, List<Integer>> hash = new HashMap<>();
        for (int i = 0; i < nums.size(); i++) {
            hash.computeIfAbsent(nums.get(i), x -> new ArrayList<>()).add(i);
        }
        
        int ans = 0;
        
        for (var kv: hash.entrySet()) {
            List<Integer> arr = kv.getValue();
            
            int i = 0;
            for (int j = 0; j < arr.size(); j++) {
                // arr.get(j) - arr.get(i) + 1 为区间长度
                // (j - i + 1) 为同值的长度
                while (i <= j && (arr.get(j) - arr.get(i) + 1 - (j - i + 1)) > k) {
                    i++;
                }
                ans = Math.max(ans, (j - i + 1));
            }
            
        }
        
        return ans;
        
    }
    
}
```

----

# 写在最后

不是因为这种"爱"太伟大，而是这种"爱"不配。

![image.png](https://pic.leetcode.cn/1692503259-EodsHK-image.png)
