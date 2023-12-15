# 第 368 场周赛 解题报告 | 珂学家 | 玄学赛 + 候选集优化枚举 + DP



---

# 前言

![image.png](https://pic.leetcode.cn/1697950833-uiXelX-image.png)

我已经受够这里了

受够这种人生了

来世请让我做东京的帅哥吧

---

# 整体评价

称之为玄学赛吧，对我而言。 T3实在没辙，觉得没有二分阶段性，就试试枚举，这边用了一个小优化，减少了候选集大小，感觉很玄学。

T4 更像一道综合的DP题，复杂度有点心虚，能过大概率全拜Java这门神奇的天选语言了。

---
## [T1. 元素和最小的山形三元组 I]()

同T2一起讲

---

## [T2. 元素和最小的山形三元组 II]()

现在前后缀拆解技巧，已经成为基本操作。

```java
class Solution {
    public int minimumSum(int[] nums) {
        int n = nums.length;
        int[] pre = new int[n];
        int[] suf = new int[n];
        
        int inf = Integer.MAX_VALUE / 2;
        pre[0] = inf;
        for (int i = 1; i < n; i++) {
            pre[i] = Math.min(pre[i - 1], nums[i - 1]);
        }
        suf[n - 1] = inf;
        for (int i = n - 2; i >= 0; i--) {
            suf[i] = Math.min(suf[i + 1], nums[i + 1]);
        }
        
        int ans = inf;
        for (int i = 1; i < n - 1; i++) {
            if (nums[i] > pre[i] && nums[i] > suf[i]) {
                ans = Math.min(ans, pre[i] + nums[i] + suf[i]);
            }
        }
        
        return ans == inf ? -1 : ans;
    }
}
```
---

## [T3. 合法分组的最少组数](https://leetcode.cn/contest/weekly-contest-368/problems/minimum-number-of-groups-to-create-a-valid-assignment/)

思维题，我想不到更好的解法，就谈谈我的玄学解

$$枚举$$

因为所有的分组都要满足

可以找到几个优化点

- 找到最小长度的组，从这边从大到小枚举 (容易想到)

- 依据某组的长度，对其合理性的划分长度，提取候选集 (需要想一想)

如果这个候选集大小和因子数相关，那这个候选集合可以从

$$n 降到 \sqrt n$$


这样的话，$O(n * \sqrt{n})的时间复杂度可以接受$


这边还有一个检测公式

$$ a \% b ==  0\ ||\ a / b + a \% b \ge b - 1 $$

$$ a是组长度，b是拆分的长度 $$

```java
class Solution {
    public int minGroupsForValidAssignment(int[] nums) {

        Map<Integer, Integer> hash = new HashMap<>();
        for (int v: nums) {
            hash.merge(v, 1, Integer::sum);
        }
        // 最小的长度
        int mz = nums.length;
        Map<Integer, Integer> lenMap = new HashMap();
        for (var kv: hash.entrySet()) {
            lenMap.merge(kv.getValue(), 1, Integer::sum);
            mz = Math.min(mz, kv.getValue());
        }

        // 所有组
        List<Map.Entry<Integer, Integer>> lists = lenMap.entrySet().stream().toList();

        // 从最小的长度，挑选其或选集 (优化点)
        List<Integer> valids = new ArrayList<>();
        for (int j = mz + 1; j >= 1; j--) {
            if (mz % j == 0 || (mz / j + mz % j) >= j - 1) {
                valids.add(j);
            }
        }
        
        // 从大到小枚举 (优化点)
        for (int j : valids) {
            int res = 0;
            boolean ok = true;
            for (var kv: lists) {
                int k = kv.getKey();
                if (k % j == 0 || (k / j + k % j >= j - 1)) {
                    res += ((k + j - 1) / j) * kv.getValue();
                } else {
                    ok = false;
                    break;
                }
            }
            if (ok) return res;
        }

        return 0;
    }
}
```

时间复杂度分析 $O(n)$, 分组和分组长度，互相制约了，难在时间复杂度上分析。

---

## [T4. 得到 K 个半回文串的最少修改次数](https://leetcode.cn/contest/weekly-contest-368/problems/minimum-changes-to-make-k-semi-palindromes/)

题意有点绕，感觉有些抗拒

一眼能想到思路，有点后怕


令opt[i][j]为前i组，以j结尾的状态，其值为最小代价

则转移方程为

$$opt[i + 1][t] = \min_{j\in[0, t - 1]} {(opt[i][j] + cost(j+1, t))}$$

$cost(i， j)定义为子字符s[i:j]的半回文串代价$

这边需要枚举长度的因子，然后挑选代价最小的。

$$ 1 \le d \lt (j - i + 1), d\ |\ (j - i + 1)$$

$$ min_{d} (\sum_{t=0}^{t=d-1} f (s[i+t:j:d])) $$

$$ f(s) 为 回文串最小修改函数 $$

最后感谢下，Java这个力扣天选之子。

```java
class Solution {

    // 提前计算200以内的因子
    static List<Integer>[]g = new List[201];
    static { 
        Arrays.setAll(g, x -> new ArrayList<>());
        for (int i = 1; i <= 200; i++) {
            for (int j = 1; j < i; j++) {
                if (i % j == 0) g[i].add(j);
            }
        }
    }

    int inf = Integer.MAX_VALUE / 10;

    Integer[][] memo;
    int compute(char[] str, int s, int e) {
        int l = e - s + 1;
        if (l == 1) {
            return inf;
        }

        if (memo[s][e] != null) {
            return memo[s][e];
        }

        // 枚举因子，然后计算取代价最小
        int ans = l;
        for (int v: g[l]) {
            List<Integer>[] groups = new List[v];
            Arrays.setAll(groups, x -> new ArrayList<>());
            for (int j = s; j <= e; j++) {
                groups[(j - s) % v].add(str[j] - '0');
            }

            int acc = 0;
            for (int j = 0; j < v; j++) {
                List<Integer> arr = groups[j];
                for (int t1 = 0, t2 = arr.size() - 1; t1 < t2; t1++, t2--) {
                    if (arr.get(t1).intValue() != arr.get(t2).intValue()) {
                        acc++;
                    }
                }
            }
            ans = Math.min(ans, acc);
        }

        return memo[s][e] = ans;
    }

    public int minimumChanges(String s, int k) {

        char[] str = s.toCharArray();
        int n = s.length();

        memo = new Integer[n][n];

        int[][] opt = new int[k][n];
        for (int i = 0; i < k; i++) {
            Arrays.fill(opt[i], inf);
        }

        for (int i = 0; i < n; i++) {
            opt[0][i] = compute(str, 0, i);
        }

        for (int i = 0; i < k - 1; i++) {
            for (int j = 0; j < n; j++) {
                if (opt[i][j] == inf) continue;
                for (int t = j + 2; t < n; t++) {
                    opt[i + 1][t] = Math.min(opt[i + 1][t], opt[i][j] + compute(str, j + 1, t));
                }
            }
        }

        return opt[k - 1][n - 1];

    }

}
```

时间复杂度为$O(n^3log(n))$


---

# 写在最后

请问.....我在哪里遇见过你？

我也是。


![image.png](https://pic.leetcode.cn/1697951323-ByMJrE-image.png)



