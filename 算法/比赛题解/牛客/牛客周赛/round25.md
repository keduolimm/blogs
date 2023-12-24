
# 牛客周赛 Round 25 解题报告 | 珂学家 | 猜猜乐 + 换根

---

# 前言

---
# 整体评价

思维场吧，T3印象深刻，其实我不会做，我就是猜的，到现在都不知道怎么过的，惭愧。

T4是换根模板题，也就这样了。

---
## [A. 小红购物](https://ac.nowcoder.com/acm/contest/72266/A)

思路: 模拟

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        char[] str = sc.next().toCharArray();

        long res = 0;
        for (int i = 0; i< n; i++) {
            if (str[i] == 'T') {
                res += arr[i];
            } else {
                res += Math.max(5, arr[i] / 100);
            }
        }
        System.out.println(res);
    }
}
```

---

## [B. 小红吃桃](https://ac.nowcoder.com/acm/contest/72266/B)

思路: 贪心

算思维题吧，就是大的和大的组合，小的和小的组合，这样整体一定是最大，乘积效应吧

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long mod = (long)1e9 + 7;

        long[] a = new long[n];
        for (int i = 0; i < n ;i++) {
            a[i] = sc.nextInt();
        }
        long[] b = new long[n];
        for (int i = 0; i < n ;i++) {
            b[i] = sc.nextInt();
        }

        long r1 = 1, r2 = 1;
        for (int i = 0; i < n; i++) {
            if (a[i] >= b[i]) {
                r1 = r1 * a[i] % mod;
                r2 = r2 * b[i] % mod;
            } else {
                r1 = r1 * b[i] % mod;
                r2 = r2 * a[i] % mod;
            }
        }

        System.out.println((r1 + r2) % mod);
    }
}

```

----

## [C. 小红的踏前斩](https://ac.nowcoder.com/acm/contest/72266/C)

思路: 贪心

其实我更愿意是DP，但是值域真的太大的。

所以从哪里切入呢？

一开始从开头无脑贪，WA

然后想是不是从峰值点，处理，感觉收敛太慢了。

后来，想了想，要不逆向思维，从后往前贪心，哈哈，蒙对了。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
        }

        // 逆向贪心
        long res = 0;
        for (int i = n - 1; i >= 0; i--) {
            if (i >= 2) {
                long d1 = arr[i] / 3, d2 = arr[i - 1] / 2, d3 = arr[i - 2];
                long md = Math.min(d1, Math.min(d2, d3));
                res += md * 5l;

                arr[i] -= 3 * md;
                arr[i - 1] -= md * 2l;
                arr[i - 2] -= md;
            }
            res += arr[i];
        }

        System.out.println(res);
    }
    
}
```

---

## [D. 小红树](https://ac.nowcoder.com/acm/contest/72266/D)

思路：换根DP

从换根的角度出发，然后在每条边上，处理边两侧的连通数差值

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {

    // 换根板子
    static class Solution {
        int n;
        List<Integer> []g;
        char[] colors;

        long[] dp1;
        long gAns = 0;

        long solve(int n, List<Integer> []g, char[] colors) {
            this.g = g;
            this.n = n;
            this.colors = colors;

            dp1 = new long[n];
            dfs1(0, -1);
            dfs2(0, -1, 0);
            return gAns;
        }

        void dfs1(int u, int fa) {
            int res = 1;
            for (int v: g[u]) {
                if (v == fa) continue;
                dfs1(v, u);
                if (colors[u] == colors[v]) {
                    res += dp1[v] - 1;
                } else {
                    res += dp1[v];
                }
            }
            dp1[u] = res;
        }

        void dfs2(int u, int fa, long w) {
            long tmp = dp1[u];
            if (fa != -1) {
                if (colors[u] ==  colors[fa]) {
                    tmp += w - 1;
                } else {
                    tmp += w;
                }
            }
            for (int v: g[u]) {
                if (v == fa) continue;
                long ctmp = tmp;
                if (colors[u] == colors[v]) {
                    ctmp -= (dp1[v] - 1);
                } else {
                    ctmp -= dp1[v];
                }
                gAns += Math.abs(ctmp - dp1[v]);
                dfs2(v, u, ctmp);
            }
        }
    }


    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        char[] str = sc.next().toCharArray();
        List<Integer>[]g = new List[n];
        Arrays.setAll(g, x->new ArrayList<>());

        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt() - 1, v = sc.nextInt() - 1;
            g[u].add(v);
            g[v].add(u);
        }

        Solution solution = new Solution();
        long res = solution.solve(n, g, str);
        System.out.println(res);
    }

}

```

---

# 写在最后

