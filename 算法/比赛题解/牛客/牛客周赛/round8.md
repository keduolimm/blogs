# 牛客周赛 Round 8 解题报告 | 珂学家 | 构造 + 树形DP

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230820/446702330_1692537700257/D2B5CA33BD970F64A6301FA75AE2EB22)


人工智能究竟能不能拥有和人一样的“爱”。

看完这本书的我觉得，这种爱，人工智能不应该去渴求拥有。

---

# 整体评价

原题场吧，开赛前就直言不讳说是来自美团的笔试题。

整体还是简单，D这个树形DP不错，可能有段时间没写树形DP题。

---

## [A. 小美的排列询问](https://ac.nowcoder.com/acm/contest/63585/A)

简单题，线性遍历即可。

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
        int u = sc.nextInt(), v = sc.nextInt();
        boolean f = false;
        for (int i = 0; !f && i < n - 1; i++) {
            if (u == arr[i] && v == arr[i + 1]) {
                f = true;
            } else if (v == arr[i] && u == arr[i + 1]) {
                f = true;
            }
        }
        System.out.println(f ? "Yes" : "No");
    }

}
```

---

## [B. 小美走公路](https://ac.nowcoder.com/acm/contest/63585/B)

环形结构和距离值，稍微麻烦些

本质就是

> min(两点之间的距离，反向两点之间的距离)

利用前缀和进行加速

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        long sum = 0;
        // 前缀和
        long[] prev = new long[n + 1];
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            prev[i + 1] = prev[i] + arr[i];
            sum += arr[i];
        }

        int u = sc.nextInt() - 1, v = sc.nextInt() - 1;
        if (u > v) {
            // swap
            int t = u; u = v; v = t;
        }

        // 普通区间前缀和
        long r1 = prev[v] - prev[u];
        // 环形结构(跨1-N)
        long r2 = sum - r1;

        System.out.println(Math.min(r1, r2));
    }
    
}
```

---

## [C. 小美的排列构造](https://ac.nowcoder.com/acm/contest/63585/C)

就是让最大的和最小的相邻，最大和次大间接相邻，感觉这个构造最使得最大最小差值最小。

即如下构造 $1,n,2,n-1,3,n-2,.....$

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        int ptr = 1;
        for (int i = 0; i < n; i += 2) {
            arr[i] = ptr++;
        }
        for (int i = n - 1; i >= 0; i--) {
            if (arr[i] == 0) {
                arr[i] = ptr++;
            }
        }
        System.out.println(Arrays.stream(arr).mapToObj(String::valueOf).collect(Collectors.joining(" ")));
    }

}
```

---

## [D. 小美的树上染色](https://ac.nowcoder.com/acm/contest/63585/D)

### 1. 树形DP解法

树形DP的好题

对于每个节点，引入0,1状态(表示不染色，染色)

那每个子树的根节点u, s为u的儿子节点集合

![alt](https://uploadfiles.nowcoder.com/images/20230821/446702330_1692581208937/D2B5CA33BD970F64A6301FA75AE2EB22)


```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {

    static class Solution {
        int[] ws;
        List<Integer> []g;
        int[][] dp;

        int solve(int n, int[] ws, List<Integer>[] g) {
            this.g = g;
            this.dp = new int[n + 1][2];
            this.ws = ws;
            dfs(1, -1);
            return Math.max(dp[1][0], dp[1][1]);
        }

        void dfs(int u, int fa) {
            int[] res = new int[] {0, 0};
            for (int v: g[u]) {
                if (v == fa) {
                    continue;
                }
                dfs(v, u);
                res[0] += Math.max(dp[v][0], dp[v][1]);
            }

            for (int v: g[u]) {
                if (v == fa) continue;
                int x2 = res[0] - Math.max(dp[v][0], dp[v][1]);

                long rr = (long)ws[u] *ws[v];
                long r = (long)Math.sqrt(rr);
                if (r * r == rr) {
                    res[1] = Math.max(2 + x2 + dp[v][0], res[1]);
                }
            }
            dp[u][0] = res[0];
            dp[u][1] = res[1];
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] ws = new int[n + 1];
        for (int i = 1; i <= n ;i++) {
            ws[i] = sc.nextInt();
        }
        List<Integer>[]g = new List[n + 1];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            g[u].add(v);
            g[v].add(u);
        }

        Solution solution = new Solution();
        int res = solution.solve(n, ws, g);
        System.out.println(res);
    }

}
```

---

### 2. 无权二分图最大匹配

看到评论区有大佬，提到了这个方法，所以补充一下。

匈牙利算法，其时间复杂度$O(V*E)$, 树的节点N，边N-1，理论会达到$O(10^{10})$。

感觉还是测试数据偏随机，完全平方数限制很强，导致时间复杂度骤降。

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {

    // 无权二分图最大匹配算法 O(VE)
    static boolean match(int u, int[] link, boolean[] used, List<Integer> []g) {
        for (int v: g[u]) {
            if (used[v]) continue;
            used[v] = true;
            if (link[v] == 0 || match(link[v], link, used, g)) {
                link[u] = v;
                link[v] = u;
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] ws = new int[n + 1];
        for (int i = 1; i <= n ;i++) {
            ws[i] = sc.nextInt();
        }
        List<Integer>[]g = new List[n + 1];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            long rr = (long)ws[u] * ws[v];
            long r = (long)Math.sqrt(rr);
            if (r * r == rr) {
                g[u].add(v);
                g[v].add(u);
            }
        }

        int ans = 0;
        int[] link = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            if (link[i] != 0) continue;
            boolean[] used = new boolean[n + 1];
            used[i] = true;
            if (match(i, link, used, g)) {
                // 找到一条增广路径
                ans++;
            }
        }

        System.out.println(ans * 2);
    }

}

```

注：记得替换快读快写.

----

# 写在最后

不是因为这种"爱"太伟大，而是这种"爱"不配。

![alt](https://uploadfiles.nowcoder.com/images/20230820/446702330_1692537721445/D2B5CA33BD970F64A6301FA75AE2EB22)

