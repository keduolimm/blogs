
# 第 388 场周赛 解题报告 | 珂学家 | 转移优化的DP + 可撤销的字典树

---
# 前言

![image.png](https://pic.leetcode.cn/1710045556-DGjRLX-image.png)

---

# 整体评价

T3落笔就想Trie，但是看了下数据规模小，就打算莽一下。T4蛮难的, 感觉需要数据结构优化的DP，但是推下公式，这里面有简化转移的技巧在。

---

## [A. 重新分装苹果](https://leetcode.cn/contest/weekly-contest-388/problems/apple-redistribution-into-boxes/)

思路: 贪心

```java []
class Solution {
    public int minimumBoxes(int[] apple, int[] capacity) {
        int s = Arrays.stream(apple).sum();
        
        // 把int转换为Integer, 方便Arrays.sort 逆序排序
        Integer[] arr = Arrays.stream(capacity).mapToObj(x -> x).toArray(Integer[]::new);
        Arrays.sort(arr, (a, b) -> b - a);
        
        for (int i = 0; i < arr.length; i++) {
            if ((s -= arr[i]) <= 0) {
                return i+ 1;
            }   
        }
        
        // unreachable
        return -1;
    }
}
```

---
## [B. 幸福值最大化的选择方案](https://leetcode.cn/contest/weekly-contest-388/problems/maximize-happiness-of-selected-children/)

思路: 贪心

从大到小贪, 结果不会变差。

```java []
class Solution {
    public long maximumHappinessSum(int[] happiness, int k) {
        Integer[] hs = Arrays.stream(happiness).mapToObj(x -> x).toArray(Integer[]::new);
        Arrays.sort(hs, (a, b) -> b - a);
        
        long sum = 0;
        for (int i = 0; i < k; i ++) {
            sum += Math.max(0, hs[i] - i);
        }
        return sum;
    }
}
```

---
## [C. 数组中的最短非公共子字符串](https://leetcode.cn/contest/weekly-contest-388/problems/shortest-uncommon-substring-in-an-array/)

思路: 数据结构题

因为数据规模小，所以用hashmap可以来代替下。

引入如下操作
- 撤销
- 添加

```java []
class Solution {
    public String[] shortestSubstrings(String[] arr) {
        Map<String, Integer> getSub = new HashMap<>();

        for (String ss : arr) {
            for (int i = 0; i < ss.length(); i++) {
                for (int j = i + 1; j <= ss.length(); j++) {
                    String substring = ss.substring(i, j);
                    getSub.put(substring, getSub.getOrDefault(substring, 0) + 1);
                }
            }
        }

        String[] ans = new String[arr.length];
        for (int i = 0; i < arr.length; i++) {
            String ss = arr[i];
            Map<String, Integer> tmp = new HashMap<>();
            for (int j = 0; j < ss.length(); j++) {
                for (int k = j + 1; k <= ss.length(); k++) {
                    String substring = ss.substring(j, k);
                    tmp.put(substring, tmp.getOrDefault(substring, 0) + 1);
                }
            }
            String p = "";
            for (String k : tmp.keySet()) {
                if (tmp.get(k).equals(getSub.get(k))) {
                    if (p.equals("") || p.length() > k.length() || (p.length() == k.length() && p.compareTo(k) > 0)) {
                        p = k;
                    }
                }
            }
            ans[i] = p;
        }

        return ans;
    }
}
```

---

- 可撤销的字典树(补充)

```java []
    class Solution {

        public String[] shortestSubstrings(String[] arr) {

            int n = arr.length;
            Trie root = new Trie();
            for (int i = 0; i < n; i++) {
                String w = arr[i];
                for (int len = 1; len <= w.length(); len++) {
                    for (int j = 0; j + len <= w.length(); j++) {
                        root.add(w.substring(j, j + len));
                    }
                }
            }

            String[] res = new String[n];
            for (int i = 0; i < n; i++) {
                String w = arr[i];
                
                // 撤销操作
                for (int len = 1; len <= w.length(); len++) {
                    for (int j = 0; j + len <= w.length(); j++) {
                        String s = w.substring(j, j + len);
                        root.erase(s);
                    }
                }

                // 进行查询
                String ans = null;
                for (int j = 0; j < w.length(); j++) {
                    int idx = root.query(w.substring(j));
                    if (idx > 0 && (ans == null || idx <= ans.length())) {
                        String slice = w.substring(j, j + idx);
                        if (ans == null || ans.length() > idx) {
                            ans = slice;
                        } else if (ans.length() == idx && ans.compareTo(slice) > 0) {
                            ans = slice;
                        }
                    }
                }

                // 加回去
                for (int len = 1; len <= w.length(); len++) {
                    for (int j = 0; j + len <= w.length(); j++) {
                        String s = w.substring(j, j + len);
                        root.add(s);
                    }
                }
                
                res[i] = (ans == null) ? "" : ans;
            }
            return res;
        }
        
        
        static class Trie {
            Trie[] cs = new Trie[26];
            int rel = 0;

            public Trie() {
            }

            void add(String w) {
                Trie cur = this;
                for (char c: w.toCharArray()) {
                    int p = c - 'a';
                    if (cur.cs[p] == null) {
                        cur.cs[p] = new Trie();
                    }
                    cur = cur.cs[p];
                    cur.rel++;
                }
            }

            void erase(String w) {
                Trie cur = this;
                for (char c: w.toCharArray()) {
                    int p = c - 'a';
                    cur = cur.cs[p];
                    cur.rel--;
                }
            }

            int query(String w) {
                Trie cur = this;
                for (int i = 0; i < w.length(); i++) {
                    char c = w.charAt(i);
                    int p = c - 'a';
                    if (cur.cs[p] == null || cur.cs[p].rel == 0) {
                        return i + 1;
                    }
                    cur = cur.cs[p];
                }
                return 0;
            }

        }
    }

```

---
## [D. K 个不相交子数组的最大能量值](https://leetcode.cn/contest/weekly-contest-388/problems/maximum-strength-of-k-disjoint-subarrays/)

思路: 转移优化的DP

这题划分型DP还是很明显的，但是往往是$O(n^2*k)$的时间复杂度。

那这题如何优化呢？

它的切入点，其实和

$$strength = sum[1] * x - sum[2] * (x - 1) + sum[3] * (x - 2) - sum[4] * (x - 3) + ... + sum[x] * 1$$

无关

---

先按常规的dp递推思路来

定义 $dp[i][j] 为前j项数组，划分为i个子数组的最大价值$

那么其转移方程为:

$$dp[i][j] = max(dp[i][j - 1], \max_{t=0}^{t=j-1} dp[i - 1][t] + sum(nums[t+1:j]) * (k - i) * (-1)^{i+1} ) $$

$$令pre(t+1, j) = sum(nums[t+1:j])的区间和$$

$$dp[i][j] = max\{ dp[i][j - 1], \max_{t=0}^{t=j-1} dp[i - 1][t] + pre(t+1,j) * (k - i) * (-1)^{i+1} \} $$

$dp[i][j], 包含是否选择nums[j]的问题$

- 不选， 则取 $dp[i][j - 1]$
- 选, 则为$\max_{t=0}^{t=j-1} dp[i - 1][t] + pre(t+1,j) * (k - i) * (-1)^{i+1}$

这题的关键就是， 如何优化

$$\max_{t=0}^{t=j-1} dp[i - 1][t] + pre(t+1,j) * (k - i) * (-1)^{i+1}$$

如何把$O(n)$的转移代价优化为$O(1)$

$$令 s(i, j) = \max_{t=0}^{t=j-1} dp[i - 1][t] + pre(t+1,j) * (k - i) * (-1)^{i+1}$$

关键是找到 ${s(i, j+1)和s(i,j)之间的关系}$

---

$$ s(i, j+1) = \max_{t=0}^{t=j} dp[i - 1][t] + pre(t+1,j + 1) * (k - i) * (-1)^{i+1}$$


$$
= max \left\{\begin{matrix}
\max_{t=0}^{t=j-1} dp[i - 1][t] + (pre[t+1:j] + nums[j+1]) * (k - i) * (-1)^{i+1} \\ \\
dp[i-1][j] + nums[j+1] * (k - i) * (-1)^{i+1}
\end{matrix}\right.
$$


$$
= max \left\{\begin{matrix}
s(i, j) + nums[j+1] * (k - i) * (-1)^{i+1} \\ \\
dp[i-1][j] + nums[j+1] * (k - i) * (-1)^{i+1}
\end{matrix}\right.
$$

即:

$$ s(i, j+1) = max(s(i, j), dp[i-1][j]) + nums[j+1] * (k - i) * (-1)^{i+1}  $$

这样就实现了$O(1)$的转移

```java []
class Solution {
    public long maximumStrength(int[] nums, int k) {
        int n = nums.length;

        long inf = Long.MIN_VALUE / 10;
        long[][] dp = new long[k][n];
        for (int i = 0; i < k; i++) {
            Arrays.fill(dp[i], inf);
        }

        long acc = 0;
        for (int i = 0; i < n; i++) {
            acc = Math.max(acc + nums[i], nums[i]);
            if (i == 0) {
                dp[0][i] = acc * k;
            } else {
                dp[0][i] = Math.max(dp[0][i - 1], acc * k);
            }
            acc = Math.max(acc, 0);
        }

        for (int w = 1; w < k; w++) {
            if (w % 2 == 1) {
                long maxV = inf;
                for (int i = 0; i < n; i++) {
                    if (i == 0) {
                        dp[w][i] = maxV;
                    } else {
                        dp[w][i] = Math.max(dp[w][i - 1], maxV - (long)nums[i] * (k - w));
                    }
                    maxV = Math.max(dp[w - 1][i], maxV - (long)nums[i] * (k - w));
                }
            } else {
                long maxV = inf;
                for (int i = 0; i < n; i++) {
                    if (i == 0) {
                        dp[w][i] = maxV;
                    } else {
                        dp[w][i] = Math.max(dp[w][i - 1], maxV + (long)nums[i] * (k - w));
                    }
                    maxV = Math.max(dp[w - 1][i], maxV + (long)nums[i] * (k - w));
                }
            }
        }

        return Arrays.stream(dp[k - 1]).max().getAsLong();
    }
}
```

时间复杂度为$O(nk)$, 空间上可以用滚动数组优化为一维$O(n)$

---

# 写在最后

![image.png](https://pic.leetcode.cn/1710045568-VliCWe-image.png)
