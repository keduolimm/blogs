# 第 363 场周赛 解题报告 | 珂学家 | 阅读理解场 + 质因子拆解 & 类调和级数


---

# 前言

![image.png](https://pic.leetcode.cn/1694938350-yndGbe-image.png)

就让我看看吧，继承了我的力量的你的剑之神威。

---

# 整体评价

真的是阅读理解场，t3惯性认为数多机器并行，瞬间傻眼，T4也顺带看错题，太惨了。

---
## [T1. 计算 K 置位下标对应元素的和](https://leetcode.cn/contest/weekly-contest-363/problems/sum-of-values-at-indices-with-k-set-bits/)

API调用+模拟

```java
class Solution {
    public int sumIndicesWithKSetBits(List<Integer> nums, int k) {
        int ans = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (Integer.bitCount(i) == k) {
                ans += nums.get(i);
            }
        }
        return ans;
    }
}
```

关于二进制位数相关的操作，系统库更快更优雅。


---

## [T2. 让所有学生保持开心的分组方法数](https://leetcode.cn/contest/weekly-contest-363/problems/happy-students/)

枚举选取的个数，然后比对，因为是严格大于/小于，没有中间态的不确定因子。

因此选取人数 能唯一确定该序列，并确定是否满足要求。

直接枚举为 $O(n^2)$, 显然不行，但是可以借助排序来优化。


```java
class Solution {
    
    public int countWays(List<Integer> nums) {
        // 排序
        Collections.sort(nums);
        int n = nums.size();
        
        int ans = 0;
        for (int i = 0; i <= n; i++) {
            // 找到数组的边界两点，进行判定
            boolean f = true;
            if (i != n && i >= nums.get(i)) {
                f = false;
            }        
            if (i != 0 && i <= nums.get(i - 1)) {
                f = false;
            }
            if (f) ans++;
        }
        
        return ans;
    }
    
}
```

---
## T3. 最大合金数

这题的核心就是

> 所有合金都需要由同一台机器制造。

太优秀了， T_T

这样的话，枚举机器，然后每个机器二分答案即可

```java
class Solution {
    
    int solve(List<Integer> need, List<Integer> stock, List<Integer> cost, int budget) {
        int l = 0, r = 10_0000_0000;
        while (l <= r) {
            int m = l + (r - l) / 2;
            
            long money = 0;
            for (int i = 0; i < need.size(); i++) {
                long left = Math.max((long)need.get(i) * m - stock.get(i), 0);
                
                money += (long)left * cost.get(i);
            }
            if (money <= budget) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        return r;
    }
    
    public int maxNumberOfAlloys(int n, int k, int budget, List<List<Integer>> composition, List<Integer> stock, List<Integer> cost) {
        
        int ans = 0;
        for (int i = 0; i < k; i++) {
            int tmp = solve(composition.get(i), stock, cost, budget);  
            ans = Math.max(ans, tmp);
        }
        
        return ans;
        
    }
    
}
```
---

## [T4. 完全子集的最大元素和](https://leetcode.cn/contest/weekly-contest-363/problems/maximum-element-sum-of-a-complete-subset-of-indices/)

很奇怪，它case中特指索引下标，但是题面却不是，一声叹息。

而且集合允许单个元素，是真的没想到。

> 一个集合任意两元素的乘积都是一个完全平方数

如何解读，或者说提取关键点

$$ 令 z = x * y $$

$$x是最大的平方数 因子， y中每个质因子最多1个$$

那么我们把每个数$z的y$ 作为key，然后按此进行分组,

$$S=\{z_1, z_2, ..., z_n\}$$

$$S=\{x_1 * y, x_2 * y, ..., x_n * y\}$$

那集合S中的，任意$z_i, z_j$, 其乘积为

$$t=x_i * x_j * y^2$$

$x_i, x_j本身就是完全平方数$，所以t是完全平方数。

这是理论基础，后续有两种思路

### 1. 质因子拆解

对每个索引，进行质因子拆解，保留奇数个数的因子乘积作为y

```java
class Solution {
    
    int solve(int v) {
        int res = 1;
        for (int i = 2; i <= v / i; i++) {
            if (v % i == 0) {
                int cnt = 0;
                while (v % i == 0) {
                    v /= i;
                    cnt++;
                }
                // 质数奇数个
                if (cnt % 2 == 1) res *= i;
            }
        }
        // 尾巴
        if (v > 1) res *= v;
        return res;
    }

    public long maximumSum(List<Integer> nums) {
        
        int n = nums.size();        
        Map<Integer, Long> map = new HashMap<>();
        for (int i = 0; i < nums.size(); i++) {
            int k = solve(i + 1);
            map.merge(k, (long)nums.get(i), Long::sum);
        }
        
        return map.values().stream().mapToLong(Long::valueOf).max().getAsLong();
        
    }

}
```

### 2. 类调和级数

枚举$1 到 \sqrt{n}$的平方数，预处理1到n每个数的最大平方因子

```java
class Solution {
    
    static int[] gHash;
    
    static {
        // JDK8不行，static初始化非常慢
        // 预处理，类似于调和级数(复杂度远低于 nlogn)
        int n = 10000;
        gHash = new int[n + 1];
        Arrays.fill(gHash, 1);
        for (int i = 2; i <= n / i; i++) {
            int v = i * i;
            for (int j = 1; j <= n / v; j++) {
                gHash[j * v] = v;
            }
        }
    }

    public long maximumSum(List<Integer> nums) {
        
        int n = nums.size();        
        Map<Integer, Long> map = new HashMap<>();
        for (int i = 0; i < nums.size(); i++) {
            int k = (i + 1) / gHash[i + 1];
            map.merge(k, (long)nums.get(i), Long::sum);
        }
        
        return map.values().stream().mapToLong(Long::valueOf).max().getAsLong();
        
    }

}
```


---

# 写在最后

当我拔出第二把剑时，就是为了守护我所爱的人。

![image.png](https://pic.leetcode.cn/1694938410-eQESZo-image.png)
