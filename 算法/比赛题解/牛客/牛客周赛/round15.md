# 牛客周赛 Round 15 解题报告 | 珂学家 | 状态DP构造 + 树形DP

---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231016/446702330_1697390055685/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

这场T3挺有意思的，只会3维状态DP进行构造。不过这题其实是脑筋急转弯，有规律可循。

T4是经典的树形DP，从比赛来看，T3难于T4.

---
## [A. 游游的整数切割](https://ac.nowcoder.com/acm/contest/66943/A)

枚举遍历就行，需要满足前后两段其末尾的元素奇偶一致

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        
        char[] str = sc.next().toCharArray();
        
        int n = str.length;
        int ans = 0;
        // 枚举分界点
        for (int i = 0; i < n - 1; i++) {
            // 前后两段的末尾元素，同奇偶
            if ((str[i] - '0') % 2 == (str[n - 1] - '0') % 2) {
                ans++;
            }
        }
        System.out.println(ans);
    }

}
```
---
## [B. 游游的字母串](https://ac.nowcoder.com/acm/contest/66943/B)

枚举最终的那个字母

然后计算代价， 然后取最小的代价即可

这里有个绕的地方，就是如何评估 字母 x -> y的代价

> p = abs(x - y) 这是两字母的距离

> dist = min(p, 26 - p), 因为环状结构，这个是最小的代价


```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();

        int ans = Integer.MAX_VALUE / 10;
        for (int i = 0; i < 26; i++) {
            int tmp = 0;
            for (int j = 0; j < str.length; j++) {
                int p = str[j] - 'a';
                // 环状结构, 两点的距离 min(s, 26 - s), s = abs(i - p)
                tmp += Math.min(26 - Math.abs(i - p), Math.abs(i - p));
            }
            ans = Math.min(ans, tmp);
        }
        System.out.println(ans);

    }

}
```

---
## [C. 游游的问号替换](https://ac.nowcoder.com/acm/contest/66943/C)

### 1. 状态DP构造解

令 opt[i][s1][s2], i表示前i个字符， s1，s2位最后两位字符(0, 2), 其为布尔值，表示该状态可构造/不可构造

转移方程为

> opt[i][s1][s2] = opt[i - 1][s0][s1]

> 满足 s0 * 9 + s1 * 3 + s2 是偶数，且s0!=s1 && s1!= s2

所以这里的状态是3维，转移又要x3，累计复杂度为$O(3^3*n)$

当然转移的时候，需要引入一个回溯from数组，用于保存转移的源头

当然这题需要特判，n=1的时候

```java
import java.io.*;
import java.util.*;

public class Main {

    static class Solution {

        int n;
        char[] str;

        public Solution(char[] str) {
            this.str = str;
            this.n = str.length;
        }

        private int[] find(boolean[][][] opt) {
            int n = opt.length;
            for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                    if (opt[n - 1][i][j]) {
                        return new int[] {i, j};
                    }
                }
            }
            return null;
        }

        private String build(int[] xy, int[][][] from) {
            // 逆序构造解
            int t1 = xy[0], t2 = xy[1];
            StringBuilder sb = new StringBuilder();
            sb.append((char)(t2 + '0'));
            sb.append((char)(t1 + '0'));
            int m = n - 1;
            while (m >= 2) {
                int c2 = t1;
                int c1 = from[m][t1][t2];
                sb.append((char)(c1 + '0'));
                t2 = c2;
                t1 = c1;
                m--;
            }
            return sb.reverse().toString();
        }

        public String solve() {
            // 特判长度1
            if (n == 1) {
                return str[0] == '?' ? "0" : new String(str);
            }

            boolean[][][] opt = new boolean[n][3][3];
            int[][][] from = new int[n][3][3]; // 追踪来源

            // 初始化前两个元素
            for (int i = 0; i < 3; i++) {
                if (str[0] != '?' && str[0] - '0' != i) continue;
                for (int j = 0; j < 3; j++) {
                    if (str[1] != '?' && str[1] - '0' != j) continue;
                    if (i != j) {
                        opt[1][i][j] = true;
                    }
                }
            }

            for (int i = 2; i < n; i++) {
                for (int j = 0; j < 3; j++) {
                    for (int k = 0; k < 3; k++) {
                        if (j == k) continue;
                        if (!opt[i - 1][j][k]) continue;

                        int t = j * 9 + k * 3;
                        for (int d = 0; d < 3; d++) {
                            if (d == k) continue;
                            if (str[i] == '?' || str[i] - '0' == d) {
                                if ((t + d) % 2 == 0) {
                                    opt[i][k][d] = true;
                                    from[i][k][d] = j;
                                }
                            }
                        }
                    }
                }
            }

            // 判断是否存在可行解
            int[] xy = find(opt);
            if (xy == null) {
                // 说明无解
                return "-1";
            }

            return build(xy, from);

        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        Solution solution = new Solution(str);
        String res = solution.solve();
        System.out.println(res);
    }

}
```

---
### 2. 脑筋急转弯

分类讨论

- n = 1

任意字符皆可

> "0", "1", "2"

- n = 2

只要 str[0] != str[1]，

> 即 "01", "02", "10", "12", "20", "21"

- n = 3

合法的字符串只有如下几种

> "020", "101", "121", "202"

- n > 3

如果要让所有的连续3长度的子串合法，根据n=3的结论，那只能是0,2字符交叉更替的情况

综合以上，就很容易了

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static class Solution {

        public String solve(char[] str) {
            // 根据长度，找到候选集
            String[] candidates = listCandidates(str.length);
            for (String s: candidates) {
                // 进行匹配
                if (match(str, s.toCharArray())) {
                    return s;
                }
            }
            // 找不到匹配
            return "-1";
        }

        // 根据长度, 返回候选的字符串
        String[] listCandidates(int n) {
            if (n == 1) {
                return new String[] {"0", "1", "2"};
            } else if (n == 2) {
                return new String[] {"01", "02", "10", "12", "20", "21"};
            } else if (n == 3) {
                return new String[] {"101", "121", "020", "202"};
            } else {
                // 超过3长度的
                // 构造0,2交叉更替的字符串
                StringBuilder sb1 = new StringBuilder();
                StringBuilder sb2 = new StringBuilder();
                for (int j = 0; j < n; j++) {
                    if (j % 2 == 0) {
                        sb1.append("2");
                        sb2.append("0");
                    } else {
                        sb1.append("0");
                        sb2.append("2");
                    }
                }
                return new String[] {sb1.toString(), sb2.toString()};
            }
        }

        // 进行匹配操作
        boolean match(char[] str, char[] template) {
            for (int j = 0; j < str.length; j++) {
                // ? 匹配 0/1/2
                // 要么0,1,2相等
                if (str[j] != '?' && str[j] != template[j]) {
                    return false;
                }
            }
            return true;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        Solution solution = new Solution();
        System.out.println(solution.solve(str));
    }

}

```

---
### 3. DFS构造

为啥DFS构造可以呢？

它不是有1000的深度吗，就不怕TLE吗？

是因为它的分支少，而且不正确的分支路径极短，因此DFS构造是适合不过的。

```java
import java.io.*;
import java.util.*;

public class Main {

    static class Solution {

        public String solve(char[] str) {
            String r = dfs(str, 0);
            return r == null ? "-1" : r;
        }

        String dfs(char[] str, int s) {
            if (s >= str.length) {
                // 找到解了
                return new String(str);
            }

            // 保留原有的值
            char cp = str[s];
            for (int i = 0; i < 3; i++) {
                // 原字符串不是 ?, 需要过滤
                if (cp != '?' && cp - '0' != i) continue;

                str[s] = (char)(i + '0');
                if (s > 0 && str[s] == str[s - 1]) continue;

                if (s <= 1) {
                    String r = dfs(str, s + 1);
                    if (r != null) return r;
                } else {
                    int acc = (str[s - 2] - '0') * 9 + (str[s - 1] - '0') * 3 + (str[s] - '0');
                    if (acc % 2 == 1) continue;
                    String r = dfs(str, s + 1);
                    if (r != null) return r;
                }
            }
            // 恢复原有的值
            str[s] = cp;
            
            // 找不到合法的字符串
            return null;
        }

    }

    public static void main(String[] args) {    
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        
        char[] str = sc.next().toCharArray();
    
        Solution solution = new Solution();
        System.out.println(solution.solve(str));
    }

}
```


---

## [D. 游游的树上边染红](https://ac.nowcoder.com/acm/contest/66943/D)

这边和 打家劫舍3(树形DP版本）还是有区别的， 一个是点不相邻，一个是边不相邻。

其实这题核心
> 不是节点染色，还是选染边(不能相邻)

因此对于每个节点, 引入两个状态(0:可被选，1:不可被选)

设计状态 dp[u][s], u为节点, s为0,1状态

状态转移

> dp[u][0] = 累加和 ( dp[v][0], dp[v][1] )， v为u的子节点

> dp[u][1] = max ( dp[u][0] - max(dp[v][0], dp[v][1]) + dp[v][0] + w(u, v) )

因为状态1，它只能选择一个子节点，两者构建一个染红色的边。

```java
import java.io.*;
import java.util.*;

public class Main {

    static class Solution {

        int n;
        List<long[]> []g;

        long[][] dp;

        // 简单的树形DP
        long solve(int n, List<long[]> []g) {
            this.n = n;
            this.g = g;
            dp = new long[n][2];
            dfs(0, -1);
            return Math.max(dp[0][0], dp[0][1]);
        }

        void dfs(int u, int fa) {
            long x1 = 0, x2 = Long.MIN_VALUE;

        
            for (long[] e: g[u]) {
                int v = (int)e[0];
                if (v == fa) continue;

                dfs(v, u);

                x1 += Math.max(dp[v][0], dp[v][1]);
                x2 = Math.max(x2, -Math.max(dp[v][0], dp[v][1]) + dp[v][0] + e[1]);
            }

            dp[u][0] = x1;
            dp[u][1] = x1 + x2;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        List<long[]> []g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());
        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt() - 1, v = sc.nextInt() - 1;
            long w = sc.nextInt();
            g[u].add(new long[] {v, w});
            g[v].add(new long[] {u, w});
        }

        Solution solution = new Solution();
        long res = solution.solve(n , g);
        System.out.println(res);
    }

}

```


---
# 写在最后
![alt](https://uploadfiles.nowcoder.com/images/20231016/446702330_1697390071574/D2B5CA33BD970F64A6301FA75AE2EB22)




