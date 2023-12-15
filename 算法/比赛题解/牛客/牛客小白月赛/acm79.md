
# 牛客小白月赛79 解题报告 | 珂学家 | 欧拉降幂 + 0-1背包 + 树形DP

---

# 前言


![alt](https://uploadfiles.nowcoder.com/images/20231026/446702330_1698329446191/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

很侧重思维的一场小白月赛，后几题都出的特别用心，特别巧妙。

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 数位dp？](https://ac.nowcoder.com/acm/contest/66877/A)

贪心， 就是逆序寻找到第一个非奇数，这一段就是最小操作数

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        int ans = 0;
        for (int i = str.length - 1; i >= 0; i--) {
            int p = str[i] - '0';
            if (p % 2 == 0) {
                break;
            } else {
                ans = (str.length - i);
            }
        }
        System.out.println(ans);
    }

}
```


---
## [B. 灵异背包？](https://ac.nowcoder.com/acm/contest/66877/B)


因为偶数一定可以，所以只要分类讨论下奇数即可。

- 奇数的个数为奇数，则去掉最小的奇数。
- 奇数的个数为偶数，则全加

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        long acc1 = 0;
        long acc2 = 0;
        int cnt1 = 0;
        int mv = Integer.MAX_VALUE;
        for (int i = 0; i < n;i++) {
            int v = sc.nextInt();
            if (v % 2 == 0) {
                // 偶数和
                acc2 += v;
            } else {
                // 奇数和
                acc1 += v;
                cnt1++; // 记录奇数个数
                mv = Math.min(mv, v); // 维护最小的奇数
            }
        }

        if (cnt1 % 2 == 0) {
            System.out.println(acc1 + acc2);
        } else {
            System.out.println(acc1 + acc2 - mv);
        }
    }

}
```



---
## [C. mex和gcd的乘积](https://ac.nowcoder.com/acm/contest/66877/C)

这题细细分析下，只有两种可能，一种是mex最大，一种是gcd最大

因为mex=n前提是0,1,...,n-1都存在，此时gcd必然为1

如果gcd最大，则必然mex=1，也就是包含0的子区间，这样的区间，包含0外，越小越好.

所以这题，其实挺费脑的，是道好题.

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        int ans = 0;
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            if (i > 0 && arr[i] != 0 && arr[i - 1] == 0) {
                ans = Math.max(ans, arr[i]);
            }
            if (i < n - 1 && arr[i] != 0 && arr[i + 1] == 0) {
                ans = Math.max(ans, arr[i]);
            }
        }

        // 完整的mex
        int ptr = 0;
        Set<Integer> ts = new HashSet<>();
        for (int i = 0; i < n; i++) {
            ts.add(arr[i]);
            if (arr[i] == ptr) {
                ptr++;
                while (ts.contains(ptr)) {
                    ptr++;
                }
            }
        }
        ans = Math.max(ans, ptr <= 1 ? 0 : ptr);

        System.out.println(ans);

    }

}
```

---

## [D. 2^20](https://ac.nowcoder.com/acm/contest/66877/D)

非常好的题，因为左移20位，必然满足要求，所以它最大操作数为20，这是一个非常好的上界。

另一方面，+1操作，x2操作，一定是+1操作在前，x2操作在后，不能有交叉.

反证: 如果存在 +1 可以 x2操作后

因为x2，必然引发末尾为0，此时+1构建连续的末尾0，比如不如 x2操作有效

因此这题，我们可以枚举+1的次数，然后补足x2，即可.


```java
import java.io.*;
import java.util.*;

public class Main {

    static int evaluate(long v, int add) {
        for (int i = 0; i < add; i++) v++;
        if ((v & 0x0fffff) == 0) return add;
        for (int i = 0; i < 20; i++) {
            v = v << 1;
            if ((v & 0x0fffff) == 0) return add + i + 1;
        }
        // unreachable
        return 20;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int t = sc.nextInt();
        while (t-- > 0) {
            int n = sc.nextInt();

            int ans = 20;
            for (int i = 0; i <= 20; i++) {
                ans = Math.min(ans, evaluate(n, i));
            }
            System.out.println(ans);
        }
    }

}

```

---
## [E. 重生之我是QQ邮箱](https://ac.nowcoder.com/acm/contest/66877/E)

因为只算最末尾的7个字符串，且只有6个按钮

第一按对的概率为1/6

总概率为 p = (1/6)^7

那期望为 E = n / p = n * 6^7

这题 长途大佬 出的很贼， 引入了很多干扰项

包括且不限于

> 小数四舍五入取整

> 如果无穷大，则返回 -1


根据题意，最终的结尾为

> E^(2^m)

因为只取个位数，而个位数不受其他位数影响, 因此可以期望E提前 mod 10

> ((E % 10) ^ (2^m)) % 10

这个形态就是 欧拉降幂

![alt](https://uploadfiles.nowcoder.com/images/20231021/446702330_1697844260077/D2B5CA33BD970F64A6301FA75AE2EB22)

根据扩展欧拉定律

这边的p=10，其欧拉函数值10*(5-1)/5*(2-1)/2=4

而 b = 2^m

然后分类讨论即可

1. m=0,1

2. m>=2

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int m = sc.nextInt();

        if (n <= 6) {
            System.out.println(-1);
        } else {
            int e = n % 10;
            for (int i = 0; i < 7; i++) {
                e = e * 6 % 10;
            }

            long r = Euler.euler(e, 2, m, 10);
            System.out.println(r);
        }
    }

    static class Euler {

        static long gcd(long a, long b) {
            return b == 0 ? a : gcd(b, a % b);
        }

        static long euler(long a, long b, long c, long p) {
            long fp = phi(p);
            long g = gcd(a, p);
            if (g == 1) {
                return ksm(a, ksm(b, c, fp), p);
            } else {
                double e = (Math.log(fp) / Math.log(b));
                if (c < e) {
                    return ksm(a, (long)Math.pow(b, c), p);
                } else {
                    return ksm(a, ksm(b, c, fp) + fp, p);
                }
            }
        }

        static long euler(long a, long b, long p) {
            long fp = phi(p);
            long g = gcd(a, p);
            if (g == 1) {
                return ksm(a, b % fp, p);
            } else {
                if (b < fp) {
                    return ksm(a, b, p);
                } else {
                    return ksm(a, b % fp + fp, p);
                }
            }
        }

        static long phi(long n) {
            long r = n;
            for (int i = 2; i <= n / i; ++i) {
                if (n % i == 0) {
                    r = r / i * (i - 1);
                    while (n % i == 0) {
                        n /= i;
                    }
                }
            }
            if (n > 1) {
                r = r / n * (n - 1);
            }
            return r;
        }

        static long ksm(long b, long v, long mod) {
            long r = 1;
            while (v > 0) {
                if (v % 2 == 1) {
                    r = r * b % mod;
                }
                b = b * b % mod;
                v /= 2;
            }
            return r;
        }

    }

}
```



---

## [F. 是牛牛还是狗勾](https://ac.nowcoder.com/acm/contest/66877/F)

一眼0-1背包

但是时间复杂度为 O(N*V)

但是这边N=10^6, V=10^3, 最大复杂度 10^9

显然直接做，是不行的

但是这题有个特例，如果N>=1001, 根据鹊巢原理，根据前缀和，必然存在2个同余(1000)相等。

> 也就是N>=1001必然有解

所以如果依旧使用 0-1背包算法

只要找到任意解就行，(最多1001项)

和 长途大佬 的解法，区别在于 不用分类讨论(n<=1000, n>1000)

当然这里还有一个难点，就是, *回溯构造*

注意java要用快读

```java
import java.io.*;
import java.util.*;
import java.util.stream.*;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        int t = sc.nextInt();
        while (t-- > 0) {
            int n = sc.nextInt();

            long acc = 0;
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
                acc += arr[i];
            }

            boolean[] dp = new boolean[1001];
            dp[0] = true;
            int[] from = new int[1001];
            Arrays.fill(from, -1);

            // n > 1000, 前缀和鹊巢原理必然有解，思路很赞
            for (int i = 0; i < n; i++) {
                int v = arr[i];

                boolean[] dp2 = dp.clone();
                for (int j = 0; j <= 1000; j++) {
                    if (!dp[j]) continue;
                    // <=1000的处理
                    if (j + v <= 1000 && !dp2[j + v]) {
                        dp2[j + v] = true;
                        from[j + v] = i;
                    }
                    // > 1000需要取余
                    if (j + v > 1000 && !dp2[j + v - 1000]) {
                        dp2[j + v - 1000] = true;
                        from[j + v - 1000] = i;
                    }
                }
                dp = dp2;
                if (dp[1000]) {
                    // 找到了，就提前推出了
                    break;
                }
            }

            if (!dp[1000]) {
                System.out.println(-1);
            } else {
                List<Integer> res = new ArrayList<>();
                int id = 1000;
                while (from[id] != -1) {
                    res.add(from[id] + 1);
                    id = (id - arr[from[id]] + 1000) % 1000;
                }

                System.out.println(acc % 1000);
                System.out.print(res.size() + " ");
                System.out.println(res.stream().map(String::valueOf).collect(Collectors.joining(" ")));
            }

        }

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}
```





---
## [G 魔法树](https://ac.nowcoder.com/acm/contest/66877/G)

这题第一感觉就是树形DP

但是这里面的状态蛮多的，不太好下手。

其实可以分情况讨论

- 全是奇数
- 全是偶数

这样的话，dfs中需要维护的状态反而少了。

还有一点就是，这题的核心是边

> 也就是边的选择/不选决定了方案数

> 只有理解这点，才能理解下面的转移方程


1. 全是奇数连通分量

每个节点，定义如下两个状态(s1, s2)

- s1表示包含该节点的连通图 累加和为 奇数

- s2表示包含该节点的连通图 累加和为 偶数

那状态转移为

s1' = s1 * t1 + s1 * t2 + s2 * t1;

s2' = s1 * t1 + s2 * t1 + s2 * t2;

---

2. 全是偶数的连通分量

每个节点，定义如下两个状态(k1, k2)

- k1表示包含该节点的连通图 累加和为 奇数

- k2表示包含该节点的连通图 累加和为 偶数

那状态转移为

k1' = k1 * t2 * 2 + k2 * t1;

k2' = k1 * t1 + k2 * t2 * 2;


所以最终结果为

> s1 + k2

```java
import java.io.*;
import java.util.*;

public class Main {

    static class Solution {

        static long mod = 998244353l;

        int n;
        int[] ws;
        List<Integer> []g;

        long solve(int n, int[] ws, List<Integer> []g) {
            this.n = n;
            this.ws = ws;
            this.g = g;

            long[] r1 = dfs1(0, -1);

            long[] r2 = dfs2(0, -1);

            return (r1[0] + r2[1]) % mod;
        }

        // 树形DP
        long[] dfs1(int u, int fa) {
            long x1 = 0, x2 = 0;

            if (ws[u] % 2 == 0) x2 = 1;
            else x1 = 1;

            for (int v: g[u]) {
                if (v == fa) continue;

                long[] cs = dfs1(v, u);
                long t1 = cs[0], t2 = cs[1];
                
                long x3 = x1 * t1 % mod + x1 * t2 % mod + x2 * t1 % mod;
                long x4 = x1 * t1 % mod + x2 * t1 % mod + x2 * t2 % mod;

                x1 = x3 % mod;
                x2 = x4 % mod;
            }

            return new long[] {x1, x2};
        }

        long[] dfs2(int u, int fa) {
            long x1 = 0, x2 = 0;

            if (ws[u] % 2 == 0) x2 = 1;
            else x1 = 1;

            for (int v: g[u]) {
                if (v == fa) continue;

                long[] cs = dfs2(v, u);
                long t1 = cs[0], t2 = cs[1];

                long x3 = x1 * t2 % mod * 2 % mod + x2 * t1 % mod;
                long x4 = x1 * t1 % mod + x2 * t2 % mod * 2 % mod;

                x1 = x3 % mod;
                x2 = x4 % mod;
            }

            return new long[] {x1, x2};
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int[] ws = new int[n];
        for (int i = 0 ;i < n; i++) {
            ws[i] = sc.nextInt() % 2;
        }

        List<Integer>[]g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt() - 1, v = sc.nextInt() - 1;
            g[u].add(v);
            g[v].add(u);
        }

        Solution solution = new Solution();
        long r = solution.solve(n, ws, g);
        System.out.println(r);
    }
    
}
```

---
# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20231026/446702330_1698329566366/D2B5CA33BD970F64A6301FA75AE2EB22)


