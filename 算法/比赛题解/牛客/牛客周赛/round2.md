# 牛客周赛 Round 2 解题报告 | 珂学家 | 字符串hash + 打家劫舍型DP + 离线双指针

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230709/446702330_1688908603996/D2B5CA33BD970F64A6301FA75AE2EB22)

在黎明到来之前，必须有人稍微照亮黑暗。

----

# 整体评价

比赛的时候，A题用了字符串Hash，哭了。B题是经典题，C是模拟题，很怕的. D也是经典题，离散双指针，套路满满。

---
## [A. 小红的环形字符串](https://ac.nowcoder.com/acm/contest/60456/A)

因为长度为1000，所以理论上 $O(n^2)$能接受

对于这种环形，最好的处理方式: $\color{blue}复制并追加$

### 方法一: 暴力

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        String str = sc.next();
        String p = sc.next();

        int ans = 0;
        int idx = 0, n = str.length();
        str = str + str;
        while ((idx = str.indexOf(p, idx)) != -1 && idx < n) {
            ans++;
            idx++;
        }

        System.out.println(ans);

    }

}
```

### 方法二: 字符串hash

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static class StringHash {
        public static final long mod = 10_0000_0007l;
        long p = 13, h = 0, b = 1l;

        int window;
        String s;
        int cur;

        public StringHash(String s, int window, int p) {
            this.s = s;
            this.window = window;
            ready();
        }

        private void ready() {
            for (int i = 0; i < window; i++) {
                h = ((h * p) % mod + (s.charAt(i) - '0')) % mod;
                b = b * p % mod;
            }
            this.cur = window;
        }

        long hash() {
            return h;
        }

        boolean hashNext() {return this.cur < this.s.length();}
        int start() {return this.cur - this.window;}
        int end() {return this.cur;}

        public void next() {
            if (this.cur >= this.s.length()) return;
            h = (h * p % mod + s.charAt(this.cur) - '0') % mod;
            h = h - b * (s.charAt(this.cur - window) - '0');
            h = (h % mod + mod) % mod;
            this.cur++;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        String str = sc.next();
        String p = sc.next();

        StringHash phash = new StringHash(p, p.length(), 13);

        int ans = 0;
        int n = str.length();
        str = str + str;

        StringHash shash = new StringHash(str, p.length(), 13);
        if (phash.hash() == shash.hash()) {
            ans++;
        }

        while (shash.hashNext()) {
            shash.next();
            if (shash.start() >= n) break;
            if (phash.hash() == shash.hash()) {
                ans++;
            }
        }

        System.out.println(ans);

    }

}

```

---

## [B. 相邻不同数字的标记](https://ac.nowcoder.com/acm/contest/60456/B)

这题就是打家劫舍型的DP

设计状态$opt[i]$为前i个元素，累计得分最大值

状态转移为

$opt[i] = max(opt[i - 2] + arr[i] + arr[i - 1]\ if\ color[i] \ne color[i - 1] , opt[i - 1]);$


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
        char[] str = sc.next().toCharArray();

        if (n <= 1) {
            System.out.println(0);
        } else {
            long[] opt = new long[n];
            opt[0] = 0;
            opt[1] = (str[0] != str[1]) ? (arr[0] + arr[1]) : 0;

            for (int i = 2; i < n; i++) {
                opt[i] = opt[i - 1];
                if (str[i] != str[i - 1]) {
                    opt[i] = Math.max(opt[i - 2] + arr[i] + arr[i - 1], opt[i]);
                }
            }

            System.out.println(opt[n - 1]);
        }

    }
    
}
```

---

## [C. 小红的俄罗斯方块](https://ac.nowcoder.com/acm/contest/60456/C)

不用考虑消除，而且只有'L'字母，求最终每列的高度。

这题就是纯粹的模拟，以及找规律。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        // 从1开始计数
        int[] cols = new int[9];
        for (int i = 0; i < n; i++) {
            int a = sc.nextInt();
            int b = sc.nextInt();

            if (a == 0) {
                int low = Math.max(cols[b], cols[b + 1]);
                cols[b] = low + 3;
                cols[b + 1] = low + 1;
            } else if (a == 90) {
                int t2 = Math.max(cols[b + 1], cols[b + 2]);
                if (t2 >= cols[b] + 1) {
                    cols[b] = t2 + 1;
                    cols[b + 1] = t2 + 1;
                    cols[b + 2] = t2 + 1;
                } else {
                    t2 = cols[b];
                    cols[b] = t2 + 2;
                    cols[b + 1] = t2 + 2;
                    cols[b + 2] = t2 + 2;
                }
            }  else if (a == 180) {
                if (cols[b] >= cols[b + 1] + 2) {
                    int low = cols[b];
                    cols[b] = low + 1;
                    cols[b + 1] = low + 1;
                } else {
                    int low = cols[b + 1];
                    cols[b] = low + 3;
                    cols[b + 1] = low + 3;
                }
            } else if (a == 270) {
                int low = Math.max(Math.max(cols[b], cols[b + 1]), cols[b + 2]);
                cols[b] = low + 1;
                cols[b + 1] = low + 1;
                cols[b + 2] = low + 2;
            }
        }

        String res = IntStream.range(0, 9)
                .filter(x -> x > 0)
                .mapToObj(x -> String.valueOf(cols[x]))
                .collect(Collectors.joining(" "));

        System.out.println(res);

    }
    
}
```

---

## [D. 小红打怪](https://ac.nowcoder.com/acm/contest/60456/D)

经典的离线+双指针解法

预处理计算每个怪物的需要的最低血量

然后排序（从小到大），构建前缀和

离散双指针即可

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Scanner;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    // 血量计算
    public static long solve(int mh, int ma) {
        long loop = mh / 4;
        long r = mh % 4;

        if (r == 0) {
            return loop * 3l * ma - ma;
        } else {
            long x1 = loop * 3l * ma;
            if (r <= 2) {
                return x1;
            } else {
                return x1 + ma;
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long h = sc.nextLong();
        long r = sc.nextLong();

        // 计算打败怪物的最低血量
        long[] bags = new long[n];
        for (int i = 0; i < n; i++) {
            bags[i] = solve(sc.nextInt(), sc.nextInt());
        }
        Arrays.sort(bags);
        // 前缀和计算
        long[] pres = new long[n];
        for (int i = 0; i < n; i++) {
            pres[i] = (i > 0 ? pres[i - 1] : 0) + bags[i];
        }

        int q = sc.nextInt();
        int[] queries = new int[q];
        for (int i = 0; i < q; i++) {
            queries[i] = sc.nextInt();
        }

        // 离线做法
        Integer[] arr = IntStream.range(0, q).boxed().toArray(Integer[]::new);
        Arrays.sort(arr, Comparator.comparing(x -> queries[x]));

        int[] res = new int[q];
        int j = 0;
        for (int i = 0; i < q; i++) {
            int x = queries[arr[i]];
            long total = h + (long)x * r;
            while (j < bags.length && total > pres[j]) {
                j++;
            }
            res[arr[i]] = j;
        }

        System.out.println(Arrays.stream(res).mapToObj(String::valueOf)
                           .collect(Collectors.joining(" ")));

    }

}
```

---

# 写在最后

在夜空所有星星熄灭的时候，所有梦想、所有溪流，都能汇入同一片大海中，那时我们终会相见。

![alt](https://uploadfiles.nowcoder.com/images/20230709/446702330_1688908574020/D2B5CA33BD970F64A6301FA75AE2EB22)
