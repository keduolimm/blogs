# 牛客周赛 Round 4 解题报告 | 珂学家 | 数学 + 思维 + 并查集

---
# 前言
![alt](https://uploadfiles.nowcoder.com/images/20230723/446702330_1690127382685/D2B5CA33BD970F64A6301FA75AE2EB22)

剑，和茶一样，只有细细品味，才能理解它的风雅。

---
# 题解

上周的比赛相对简单，结果今天上强度了，不光题目变难了，而且题数还变多了，稍稍感觉有些吃力。

B, D偏数学，C感觉很特别，有明显的分段性，E是大模拟，有着明显的并查集痕迹。


---
## [A. 游游的字符串构造](https://ac.nowcoder.com/acm/contest/61571/A)

“you”的字符串不存在特殊性。
因此先构建k个连续的“you”， 后续添加重复的y字符即可。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int k = sc.nextInt();
        if (3 * k > n) {
            System.out.println("-1");
        } else {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < k; i++) {
                sb.append("you");
            }
            for (int i = 3 * k; i < n; i++) {
                sb.append("y");
            }
            System.out.println(sb.toString());
        }
    }

}
```

---

## [B. 游游的整数拆分](https://ac.nowcoder.com/acm/contest/61571/B)

这题的关键：***3是质数***

因为$a\times b$是3的倍数，可推导出a,b至少一个是3的倍数

分类讨论

1. n % 3 == 0，则 a，b皆为3的倍数，提取公因子3，则$3\times(a'+b') = n$， $a',b'都是正整数$，则有$(n/3-1)$种组合

2. n % 3 != 0, 则 a,b只有一个为3的倍数，则组合为 $floor(n/3)$, 因为对称性，所以为 $2 \times floor(n/3)$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long n = sc.nextLong();

        if (n % 3 == 0) {
            System.out.println(Math.max(0, (n / 3 - 1)));
        } else {
            System.out.println(n / 3 * 2);
        }
    }

}
```


---

## [C. 游游的整数操作](https://ac.nowcoder.com/acm/contest/61571/C)

这题是这场比赛，最有意思的题。

这题难就难在 $max(a_i - x, 0)$ 这个操作。

因为这题的数据规模是$10^5$, 所以一直在想怎么优化

有想过合并操作，比如连续的add x可以合并，连续的sub x也可以，但是总可以构造出操作序列，导致序列没法真正实质性压缩数量级。

那就回到被操作的数组上，首先彼此独立，无关顺序。

单独取一个数${a_i}$，执行操作，当变成负数时，就会变成中间态的0。 那可以预测比${a_i}$稍小的数，同样会变成0。这样这两个数虽然不同，但最终的结果是相同的。

因此可以明显的观察到，这个操作对不同的数而言，有明显的分段性。

即在同范围内的初始数，其最终结果是一样的。

那如何求区间内，如果区间很多怎么办，好像又没降数量级。

### 方法一：分集合模拟

实际上可以把序列分成两类
- 一类是当前最小的数
- 一类是其他数（没有变成过负数）

如果最小数是安全的（没有变成负数），则第二类肯定安全
如果最小数变成负数了，则从第二类中抽取也变成负数的部分，然后合并到第一类。

这样就可以在线性求解这个问题，最后的结果为第一类的和，以及第二类(没有负数变0）的和

比赛中，我其实是这个思路。

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static long mod = 10_0000_0007l;

    public static long wrapMod(long v) {
        return (v % mod + mod) % mod;
    }

    public static void main(String[] args) {
        // 没用快读，是因为不实用也可以过
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int k = sc.nextInt();


        TreeMap<Long, Integer> hash = new TreeMap<>();
        for (int i = 0; i < n; i++) {
            long t = sc.nextLong();
            hash.merge(t, 1, Integer::sum);
        }

        // *) 对操作进行合并
        // 这边有个归0的操作
        long delta = 0;
        List<long[]> res = new ArrayList<>();
        for (Map.Entry<Long, Integer> kv: hash.entrySet()) {
            res.add(new long[] {kv.getKey(), kv.getValue()});
        }

        // 维护两个集合
        long cur = res.get(0)[0];
        long cnt = res.get(0)[1];
        int ptr = 1;

        for (int i = 0; i < k; i++) {
            int op = sc.nextInt();
            long x = sc.nextLong();

            if (op == 1) {
                delta += x;
                cur += x;
            } else {
                delta -= x;

                if (cur - x >= 0) {
                    cur -= x;
                } else { 
                    // 这边比较特殊 （变成负数，有个合并操作）
                    cur = 0;
                    while (ptr < res.size() && res.get(ptr)[0] + delta <= 0) {
                        cnt += res.get(ptr)[1];
                        ptr++;
                    }
                }
            }
        }

        long ans = cur % mod * cnt % mod;
        for (int j = ptr; j < res.size(); j++) {
            long u = res.get(j)[0];
            long n2 = res.get(j)[1];
            ans += wrapMod(u + delta) * n2 % mod;
            ans %= mod;
        }

        System.out.println(ans);

    }
    
}
```

### 方法二: 寻找临界点

实际上，这题可以继续提炼，就是找到一个分割点，这个分割点以内都可以最终为（变过0），以外没变过。

***这一点很像，位运算NAND，寻找最后一个0的思路，因为最终都会归一。***

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static long mod = 10_0000_0007l;

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int k = sc.nextInt();

        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
        }

        int[][] ops = new int[k][2];
        
        long last = Long.MIN_VALUE;
        long delta = 0;
        for (int i = 0; i < k; i++) {
            int op = sc.nextInt();
            int x = sc.nextInt();
            if (op == 1) {
                delta += x;   
            } else {
                delta -= x;
                last = Math.max(last, -delta);
            }
            ops[i][0] = op; ops[i][1] = x;
        }

        long edgeRes = last;
        for (int i = 0; i < k; i++) {
            if (ops[i][0] == 1) {
                edgeRes += ops[i][1];
            } else {
                edgeRes = Math.max(edgeRes - ops[i][1], 0);
            }
        }
        
        long ans = 0;
        for (int i = 0; i < n; i++) {
            if (arr[i] <= last) {
                ans += edgeRes;
                ans %= mod;
            } else {
                ans += (long)arr[i] + delta;
                ans %= mod;
            }
        }
        
        System.out.println(ans);

    }
    
}
```

---
## [D. 游游的因子计算](https://ac.nowcoder.com/acm/contest/61571/D)

这题是非常典的题，就是质因子拆解，然后通过质因子反向构造因子解。

```java
import java.io.BufferedInputStream;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    // 质因子拆解，这边用treemap，主要是为了合并方便
    public static TreeMap<Integer, Integer> primes(int u) {
        TreeMap<Integer, Integer> res = new TreeMap<>();
        for (int i = 2; i <= u / i; i++) {
            if (u % i == 0) {
                int cnt = 0;
                while (u % i == 0) {
                    u /= i;
                    cnt++;
                }
                res.put(i, cnt);
            }
        }
        if (u > 1) {
            res.put(u, 1);
        }
        return res;
    }

    // dfs构造解, 时间复杂度很低
    public static void dfs(int s, List<int[]> lists, long now, TreeSet<Long> collector) {
        collector.add(now);
        if (s >= lists.size()) {
            return;
        }
        int[] cur = lists.get(s);
        for (int i = 0; i <= cur[1]; i++) {
            dfs(s + 1, lists, now, collector);
            now *= cur[0];
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int a = sc.nextInt();
        int b = sc.nextInt();

        TreeMap<Integer, Integer> p1 = primes(a);
        TreeMap<Integer, Integer> p2 = primes(b);

        for (Map.Entry<Integer, Integer> kv: p1.entrySet()) {
            p2.merge(kv.getKey(), kv.getValue(), Integer::sum);
        }

        List<int[]> lists = new ArrayList<>();
        for (Map.Entry<Integer, Integer> kv: p2.entrySet()) {
            lists.add(new int[] {kv.getKey(), kv.getValue()});
        }

        TreeSet<Long> ts = new TreeSet<>();
        dfs(0, lists, 1l, ts);

        System.out.println(ts.size());
        System.out.println(ts.stream().map(String::valueOf).collect(Collectors.joining(" ")));
    }

}
```


---

## [E. 小红的战争棋盘](https://ac.nowcoder.com/acm/contest/61571/E)

大模拟题，主要是基于并查集。

为了方便处理，这边主要是对并查集进行扩展。

大概就是这样, 模拟即可，不太容易错。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.StringTokenizer;

public class Main {

    static class Dsu {
        int n;
        int[] arr;
        int[] gz;
        String[] owner;

        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            this.gz = new int[n + 1];
            Arrays.fill(gz, 1);

            this.owner = new String[n + 1];
        }

        int find(int u) {
            if (arr[u] == 0) {
                return u;
            }
            return arr[u] = find(arr[u]);
        }

        // 定向合并
        void union(int u, int v) {
            int ai = find(u);
            int bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
                gz[bi] += gz[ai];
            }
        }

        int gsize(int u) {
            return gz[find(u)];
        }

        void set(int u, String name) {
            this.owner[find(u)] = name;
        }

        String get(int u) {
            return this.owner[find(u)];
        }

    }

    static class Hero {
        int y, x;
        String name;
    }

    // WSAD
    public static final int[][] dirs = new int[][] {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    public static int dir(String cmd) {
        if ("W".equalsIgnoreCase(cmd)) return 0;
        if ("S".equalsIgnoreCase(cmd)) return 1;
        if ("A".equalsIgnoreCase(cmd)) return 2;
        if ("D".equalsIgnoreCase(cmd)) return 3;
        return 0;
    }

    public static void main(String[] args) {

        AReader sc = new AReader();
        int n = sc.nextInt(), m = sc.nextInt();

        // *) 这个就很牛
        Map<String, Hero> hash = new HashMap<>();
        Dsu dsu = new Dsu((n + 1) * (m + 1));

        int k = sc.nextInt();
        for (int i = 0; i < k; i++) {
            String name = sc.next();
            int y = sc.nextInt(), x = sc.nextInt();

            Hero hero = new Hero();
            hero.name = name;
            hero.y = y;
            hero.x = x;

            String prev = dsu.get(y * m + x);
            if (prev == null) {
                hash.put(name, hero);
                dsu.set(y * m + x, name);
            } else {
                // 这个时候要发送从图
                if (prev.compareTo(name) < 0) {
                    hash.remove(prev);
                    hash.put(name, hero);
                    dsu.set(y * m + x, name);
                }
            }
        }

        int op = sc.nextInt();

        for (int i = 0; i < op; i++) {
            String name = sc.next();
            String cmd = sc.next();

            if (!hash.containsKey(name)) {
                System.out.println("unexisted empire.");
            } else {
                Hero hero = hash.get(name);

                int d = dir(cmd);
                int y0 = hero.y + dirs[d][0];
                int x0 = hero.x + dirs[d][1];

                if (y0 >= 1 && y0 <= n && x0 >= 1 && x0 <= m) {

                    String prev = dsu.get(y0 * m + x0);
                    if (prev == null) {
                        dsu.union(y0 * m + x0, hero.y * m + hero.x);
                        hero.y = y0; hero.x = x0;
                        System.out.println("vanquish!");
                    } else {
                        if (prev.equalsIgnoreCase(name)) {
                            hero.y = y0; hero.x = x0;
                            System.out.println("peaceful.");
                        } else {
                            Hero other = hash.get(prev);
                            // 发生battle
                            int n1 = dsu.gsize(hero.y * m + hero.x);
                            int n2 = dsu.gsize(other.y * m + other.x);

                            if (n1 > n2 || (n1 == n2 && name.compareTo(other.name) > 0)) {
                                dsu.union(other.y * m + other.x, hero.y * m + hero.x);
                                hero.y = y0; hero.x = x0;

                                hash.remove(prev);
                                System.out.println(name + " wins!");
                            } else {
                                dsu.union(hero.y * m + hero.x, other.y * m + other.x);

                                hash.remove(name);
                                System.out.println(prev + " wins!");
                            }
                        }
                    }
                } else {
                    System.out.println("out of bounds!");
                }
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

# 写在最后

若知是梦何须醒，不比真如一相会。

![alt](https://uploadfiles.nowcoder.com/images/20230723/446702330_1690127324751/D2B5CA33BD970F64A6301FA75AE2EB22)