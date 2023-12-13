# 牛客周赛 Round 16 解题报告 | 珂学家 | 俄罗斯套娃 + 最小生成树

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231022/446702330_1697980109686/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

很典的一场比赛， T3是俄罗斯套娃模型(如果n>=1e5), dp解(n<=1000)， T4是最小生成树，这边用kruskal(并查集)来构建.

---

## [A. 小美的升序数组](https://ac.nowcoder.com/acm/contest/67355/A)

模拟题，按题意要求就行

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        int[] brr = new int[n - 1];

        arr[0] = sc.nextInt();
        for (int i = 1; i < n; i++) {
            arr[i] = sc.nextInt();
            brr[i - 1] = arr[i] - arr[i - 1];
        }

        // 模拟严格递增/严格递减
        boolean f = true;
        for (int i = 1; i < n; i++) {
            if (arr[i - 1] >= arr[i]) f = false;
        }
        for (int i = 1; i < n - 1; i++) {
            if (brr[i - 1] <= brr[i]) f = false;
        }

        System.out.println(f ? "Yes" : "No");
    }

}
```

---
## [B. 小美打靶](https://ac.nowcoder.com/acm/contest/67355/B)

枚举1~11的半径长度，然后以外直接返回0，这边可以避免浮点数运算

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static int solve(int x, int y) {
        for (int r = 1; r <= 11; r++) {
            if (x * x + y * y <= r * r) return 11 - r;
        }
        return 0;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int x = sc.nextInt();
        int y = sc.nextInt();
        System.out.println(solve(x, y));
    }

}
```

---

## [C. 小美打怪](https://ac.nowcoder.com/acm/contest/67355/C)

### 二维偏序 转 LIS

非常经典的俄罗斯套娃模型题

为啥这么说呢？ 可以把血量抽象为一个套娃的长，攻击力等价为套娃的宽， 那这题等价于


> 有很多长宽不一的套娃，求最多的套娃数量 (保证下面的长宽严格大于上面的套娃).

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231026/446702330_1698334268462/D2B5CA33BD970F64A6301FA75AE2EB22){:width=480px}{:align=left}



就是二维偏序，构建最长的递增(递减)序列

- 先过滤，把不合法的怪物过滤掉
- 按血量(从大到小)，攻击力(从小到大)排序
- 二维偏序 转 一维 LIS (最长递增序列)

可以思考下，为啥要这么设定排序规则, 因为题目要求严格

如果题目不要求严格, 则按血量(从大到小)，攻击力(从大到小)排序

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int h = sc.nextInt(), a = sc.nextInt();

        int[][] ps = new int[n][2];
        for (int i = 0; i < n; i++) {
            ps[i][0] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            ps[i][1] = sc.nextInt();
        }

        // 俄罗斯套娃(这边做过滤，把不合法的小去掉)
        List<int[]> arr = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (ps[i][0] < h && ps[i][1] < a) arr.add(ps[i]);
        }

        // 经典的俄罗斯套娃
        // 第一个元素(从大到小), 第二个元素(从小到大)
        Collections.sort(arr, Comparator.comparingInt((int[] c) -> -c[0]).thenComparingInt(c -> c[1]));

        // LIS 最长递增(递减)序列
        List<Integer> lis = new ArrayList<>();
        for (int[] e: arr) {
            int l = 0, r = lis.size() - 1;
            while (l <= r) {
                int m = l + (r - l) / 2;
                if (lis.get(m) <= e[1]) {
                    r = m - 1;
                } else {
                    l = m + 1;
                }
            }
            if (l >= lis.size()) {
                lis.add(e[1]);
            } else {
                lis.set(l, e[1]);
            }
        }

        System.out.println(lis.size());
    }

}

```

时间复杂度为O(nlogn)

---

### 普通DP解法

这题也可以使用DP解法，因为n=1000，O(n^2)

这个解法更通用，也容易被接受

先排序, 按一个维度(从大到小)

然后线性DP

dp[i] = max(dp[j] + 1, 1),  j < i 且 血量/攻击力属性  都满足严格小于

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int h = sc.nextInt(), a = sc.nextInt();

        int[][] ps = new int[n][2];
        for (int i = 0; i < n; i++) {
            ps[i][0] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            ps[i][1] = sc.nextInt();
        }

        // 这边做过滤，把不合法的小去掉
        List<int[]> arr = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (ps[i][0] < h && ps[i][1] < a) arr.add(ps[i]);
        }

        Collections.sort(arr, Comparator.comparingInt(x -> -x[0]));

        int m = arr.size();

        int ans = 0;
        int inf = Integer.MIN_VALUE / 10;
        int[] dp = new int[m];
        Arrays.fill(dp, inf);
        for (int i = 0; i < m; i++) {
            dp[i] = Math.max(1, dp[i]);
            for (int j = 0; j < i; j++) {
                if (arr.get(i)[0] < arr.get(j)[0] && arr.get(i)[1] < arr.get(j)[1] && dp[i] < dp[j] + 1) {
                    dp[i] = dp[j] + 1;
                }
            }
            ans = Math.max(ans, dp[i]);
        }

        System.out.println(ans);

    }

}

```

### RMQ数据结构优化的DP

dp[i] = max(dp[j] + 1, 1),  j < i 且 血量/攻击力属性  都满足严格小于

其实根据这个式子，在结合二维偏序的问题

是可以引入 支持RMQ的数据结构，来加速DP求解

线段树可以，当然因为它的查询范围为[r, +inf], 因此也可以用特别的树状数组来构建

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static class BIT {
        int n;
        int[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }
        // 往[0, p]更新
        void update(int p, int d) {
            while (p > 0) {
                arr[p] = Math.max(arr[p], d);
                p -= p & -p;
            }
        }
        // 其实是[p, n]范围的最大值
        int query(int p) {
            int r = 0;
            while (p <= n) {
                r = Math.max(r, arr[p]);
                p += p & -p;
            }
            return r;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int h = sc.nextInt(), a = sc.nextInt();

        int[][] ps = new int[n][2];
        for (int i = 0; i < n; i++) {
            ps[i][0] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            ps[i][1] = sc.nextInt();
        }

        // 俄罗斯套娃(这边做过滤，把不合法的小去掉)
        List<int[]> arr = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (ps[i][0] < h && ps[i][1] < a) arr.add(ps[i]);
        }

        // 经典的俄罗斯套娃
        // 第一个元素(从大到小), 第二个元素(从小到大)
        Collections.sort(arr, Comparator.comparingInt((int[] x) -> -x[0]).thenComparingInt(x -> x[1]));

        Set<Integer> ts = new TreeSet<>();
        for (int[] e: arr) {
            ts.add(e[0]);
            ts.add(e[1]);
        }
        Map<Integer, Integer> idMap = new HashMap<>();
        int ptr = 0;
        for (int k: ts) {
            idMap.put(k, ++ptr);
        }

        int ans = 0;
        BIT bit = new BIT(ptr);
        for (int[] e: arr) {
            int r = bit.query(idMap.get(e[1]) + 1);
            ans = Math.max(ans, r + 1);
            bit.update(idMap.get(e[1]), r + 1);
        }

        System.out.println(ans);
    }

}

```


---
## [D. 小美的修路](https://ac.nowcoder.com/acm/contest/67355/D)

经典板子题，最小生成树，这边使用kruskal(并查集)来构建

因为必选路径的存在，可能最终的结果不见得是树，但是连通的概念不变

```java
import java.io.BufferedInputStream;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    static class Dsu {
        int n;
        int[] arr;
        int[] gz;
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            this.gz = new int[n + 1];
            Arrays.fill(gz, 1);
        }
        int find(int u) {
            if (arr[u] == 0) return u;
            return arr[u] = find(arr[u]);
        }
        void union(int u, int v) {
            int ai = find(u), bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
                gz[bi] += gz[ai];
            }
        }
        int gSize(int u) {
            return gz[find(u)];
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();

        PriorityQueue<int[]> pp = new PriorityQueue<>(Comparator.comparingInt(x -> x[2]));
        List<Integer> plans = new ArrayList<>();

        Dsu dsu = new Dsu(n);
        for (int i = 0; i < m; i++) {
            int u = sc.nextInt(), v = sc.nextInt(), w = sc.nextInt(), p = sc.nextInt();
            if (p == 1) {
                plans.add(i + 1);
                dsu.union(u, v);
            } else {
                pp.offer(new int[] {u, v, w, i + 1});
            }
        }

        // 最小生成树 kruskal
        while (dsu.gSize(1) < n && !pp.isEmpty()) {
            int[] cur = pp.poll();
            int u = cur[0], v = cur[1];
            if (dsu.find(u) != dsu.find(v)) {
                dsu.union(u, v);
                plans.add(cur[3]);
            }
        }

        if (dsu.gSize(1) != n) {
            System.out.println(-1);
        } else {
            System.out.println(plans.size());
            System.out.println(plans.stream().map(String::valueOf).collect(Collectors.joining(" ")));
        }

    }

}

```

--- 

# 写在最后

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231022/446702330_1697980133011/D2B5CA33BD970F64A6301FA75AE2EB22)
