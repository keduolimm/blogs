

# 第 383 场周赛 解题报告 | 珂学家 | Z函数/StringHash

---

# 前言


![image.png](https://pic.leetcode.cn/1707018538-qJhPLI-image.png)

谁言别后终无悔
寒月清宵绮梦回
深知身在情长在
前尘不共彩云飞


---

# 整体评价

T3是道模拟题，但是感觉题意有些晦涩，T4一眼Z函数，当然StringHash更通用些。

新年快乐, ^_^.

---

## [T1. 将单词恢复初始状态所需的最短时间 I](https://leetcode.cn/contest/weekly-contest-383/problems/ant-on-the-boundary/)

思路: 模拟

就是前缀和为0的次数

```java []
class Solution {
    public int returnToBoundaryCount(int[] nums) {
        int acc = 0;
        int res = 0;
        for (int v: nums) {
            acc += v;
            if (acc == 0) res++;
        }
        return res;
    }
}
```

---

## [T2. 将单词恢复初始状态所需的最短时间 I](https://leetcode.cn/contest/weekly-contest-383/problems/minimum-time-to-revert-word-to-initial-state-i/)

和T4一起讲

---
## [T3. 找出网格的区域平均强度](https://leetcode.cn/contest/weekly-contest-383/problems/find-the-grid-of-region-average/)

思路: 模拟

感觉题意有些晦涩，窗口比较小，可以莽些。

如果窗口大小比较大，就需要

$$二维前缀和+二维差分$$


```java []
class Solution {

    int[][] dirs = new int[][] {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    int[] evaluate(int[][] image, int ty, int tx, int limit) {
        int res = 0;
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {

                for (int k = 0; k < 4; k++) {
                    int dy = i + dirs[k][0];
                    int dx = j + dirs[k][1];
                    if (dy >= 0 && dy < 3 && dx >= 0 && dx < 3) {
                        int dist = Math.abs(image[ty+i][tx+j] - image[ty+dy][tx+dx]);
                        if (dist > limit) {
                            return new int[] {0, 0};
                        }
                    }
                }
                res += image[ty + i][tx + j];
            }
        }
        return new int[] {1, res / 9};
    }


    public int[][] resultGrid(int[][] image, int threshold) {

        int h = image.length, w = image[0].length;

        int[][] sum = new int[h][w];
        int[][] cnt = new int[h][w];

        // 预处理，枚举左上角
        for (int i = 0; i < h - 2; i++) {
            for (int j = 0; j < w - 2; j++) {
                int[] res = evaluate(image, i, j, threshold);

                for (int s = 0; s < 3; s++) {
                    for (int t = 0; t < 3; t++) {
                        sum[s+i][t+j] += res[1];
                        cnt[s+i][t+j] += res[0];
                    }
                }
            }
        }

        // 统计值
        int[][] ans = new int[h][w];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (cnt[i][j] == 0) {
                    ans[i][j] = image[i][j];
                } else {
                    ans[i][j] = sum[i][j] / cnt[i][j];
                }
            }
        }
        return ans;

    }
}
```

---

## [T4. 将单词恢复初始状态所需的最短时间 II](https://leetcode.cn/contest/weekly-contest-383/problems/minimum-time-to-revert-word-to-initial-state-ii/)

思路: Z函数/StringHash

属于思维题的范畴

$$S[i:n] == S[0:n-i]，那么该i就是满足要求的点$$

因为字符串S很大，所以利用StringHash，可以$O(N)$判定。

当然也可以转换下思维

$$S[i:n] == S[0:n-i] 本质就是求 S[i:n] 和 S[0:n]的最长前缀长度$$

这个就是属于Z函数的核心概念了。


- Z函数

```java []
class Solution {

     // z函数模板, https://oi-wiki.org/string/z-func/
     static int[] zFunction(String a) {
        char[] s = a.toCharArray();
        int n = s.length;
        int[] z = new int[n];
        z[0] = n;
        for (int i = 1, l = 0, r = 0; i < n; ++i) {
            if (i <= r && z[i - l] < r - i + 1) {
                z[i] = z[i - l];
            } else {
                z[i] = Math.max(0, r - i + 1);
                while (i + z[i] < n && s[z[i]] == s[i + z[i]]) ++z[i];
            }
            if (i + z[i] - 1 > r) {
                l = i;
                r = i + z[i] - 1;
            }
        }
        return z;
    }

    public int minimumTimeToInitialState(String word, int k) {
        int n = word.length();
        int[] arr = zFunction(word);
        for (int i = k; i < n; i += k) {
            if (arr[i] == n - i) {
                return i / k;
            }
        }
        return (n + k - 1) / k;
    }
    
}
```

- StringHash

```java []
class Solution {

    static
    class StringHash {
        char[] str;
        long p, mod;

        int n;
        long[] pre; // hash前缀和
        long[] pow; // p的幂次

        public StringHash(String s, int p, long mod) {
            this.str = s.toCharArray();
            this.p = p;
            this.mod = mod;

            this.n = str.length;
            pre = new long[n + 1];
            pow = new long[n + 1];

            pow[0] = 1;
            for (int i = 1; i <= n; i++) {
                pow[i] = pow[i - 1] * p % mod;
            }
            for (int i = 0; i < str.length; i++) {
                pre[i + 1] = (pre[i] * p % mod + str[i]) % mod;
            }
        }

        long query(int l, int r) {
            long res = pre[r + 1] - pre[l] * pow[r - l + 1] % mod;
            return (res % mod + mod) % mod;
        }

        long rotate(int l) {
            if (l < 0 || l >= str.length - 1) {
                return query(0, str.length - 1);
            } else {
                long h1 = query(0, l);
                long h2 = query(l + 1, str.length - 1);
                return (h2 * pow[l + 1] % mod + h1) % mod;
            }
        }

        static long evaluate(String s, int p, long mod) {
            long h = 0;
            for (char c: s.toCharArray()) {
                h = (h * p % mod + c) % mod;
            }
            return h;
        }
    }

    public int minimumTimeToInitialState(String word, int k) {
        int p1 = 13, p2 = 17;
        long mod = (long)1e9 + 7;
        StringHash h1 = new StringHash(word, p1, mod);
        StringHash h2 = new StringHash(word, p2, mod);

        int res = 1;
        int n = word.length();
        for (int i = k; i < n; i += k) {
            int d = n - i;
            if (h1.query(i, n - 1) == h1.query(0, d - 1)
                && h2.query(i, n - 1) == h2.query(0, d - 1)) {
                return res;
            }
            res++;
        }
        return res;
    }
}
```

---

# 写在最后


![image.png](https://pic.leetcode.cn/1707018804-JDJEZd-image.png)

