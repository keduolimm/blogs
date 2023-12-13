# 牛客周赛 Round 1 解题报告 | 珂学家 | 分类计数 + 同余DP

---

# 前言

生于生时，亡于亡刻。遵从自心，尽人之事。

---

# 整体评价

终于等来了侧重面试的比赛，而且题量刚刚好，不超纲，不涉及算法竞赛。

第一场的比赛，感觉题目出的比较典，A是简单模拟，B则是计数题，C则是贪心思路，D是经典的同余DP。

唯一吐槽的是，牛客好像当前只JDK 8, 用不了var.

---

## [A. 游游画U](https://ac.nowcoder.com/acm/contest/60245/A)

找规律题吧，就是找到底托，中间对等分开，然后往两边靠齐。

而N，则限定了最终图的长宽高，还有就是‘*’的长度

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        
        // 定义结果的画布
        char[][] res = new char[4 * n][4 * n];
        
        // 前面的3*n行
        for (int i = 0; i < 3 * n; i++) {
            Arrays.fill(res[i], '.');
            for (int j = 0; j < n; j++) {
                res[i][j] = res[i][4*n - 1 - j] = '*';
            }
        }
        
        // 最下面的n行
        for (int i = 3 * n; i < 4 * n; i++) {
            int offset = i + 1 - 3 * n;
            Arrays.fill(res[i], '.');
            for (int j = 0; j < n; j++) {
                res[i][offset + j] = res[i][4 * n - 1 - offset - j] = '*';
            }
        }
        
        // 输出内容
        for (int i = 0; i < 4 * n; i++) {
            System.out.println(new String(res[i]));
        }


    }

}
```
---

## [B. 游游的数组染色](https://ac.nowcoder.com/acm/contest/60245/B)

先按同值进行分类，然后内部按颜色进行计数

${map[value][color]}$

那最终的结果为

$\sum_{v_i} \sum_{c_j \in C} map[v_i][c_j] * (\sum_{k\in C} map[v_i][c_k] - map[v_i][c_j])$

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        char[] str = sc.next().toCharArray();

        Map<Integer, TreeMap<Character, Integer>> range = new HashMap<>();
        for (int i = 0; i < n; i++) {
            if (!range.containsKey(arr[i])) {
                range.put(arr[i], new TreeMap<>());
            }
            range.get(arr[i]).merge(str[i], 1, Integer::sum);
        }

        long ans = 0;
        for (Map.Entry<Integer, TreeMap<Character, Integer>> kv: range.entrySet()) {
            TreeMap<Character, Integer> ts = kv.getValue();
            long acc = 0;
            for (Map.Entry<Character, Integer> kv2: ts.entrySet()) {
                acc += kv2.getValue();
            }

            for (Map.Entry<Character, Integer> kv2: ts.entrySet()) {
                ans += (long)(acc - kv2.getValue()) * kv2.getValue();
            }
        }

        System.out.println(ans / 2);
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
        
    }

}

```
---

## [C. 游游的交换字符](https://ac.nowcoder.com/acm/contest/60245/C)

分类讨论，即构造1开头的串，和0开头的串，两者那个代价小。

一旦确定分头，实际上代价就定下来，重要的是如何计算。

比如当前串，以及构建好了'01...1', 你要寻找下一个‘0’， 则需要多少代价呢？

这个时候，就是从原串中，下一个‘0’之前有几个'1'呢

同理，‘1’也是统计前置几个‘0’

那这个动态有变动的统计0-1该怎么处理呢？

我们可以尝试引入 0-1 树状数组

就是有一个单独统计‘1’的位置的树状数组，一个统计‘0’的树状数组

前置的01构建完后，就删掉对应的01位置即可。


```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {

    static class BIT {
        int n;
        int[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }
        void update(int p, int d) {
            while (p <= n) {
                arr[p] += d;
                p += p & -p;
            }
        }
        int query(int p) {
            int res = 0;
            while (p > 0) {
                res += arr[p];
                p -= p & -p;
            }
            return res;
        }
    }

    // *)
    public static long solve(char[] buf, char s) {
        // 找到最近的0
        int n = buf.length;
        BIT bit0 = new BIT(n);
        BIT bit1 = new BIT(n);

        List<Integer> list0 = new ArrayList<>();
        List<Integer> list1 = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (buf[i] == '0') {
                list0.add(i + 1);
                bit0.update(i + 1, 1);
            } else {
                list1.add(i + 1);
                bit1.update(i + 1, 1);
            }
        }

        long ans = 0l;

        if (s == '0') {
            if (list0.size() < list1.size()) return Long.MAX_VALUE;

            for (int i = 0; i < n; i++) {
                if (i % 2 == 0) {
                    int idx = list0.get(i/2);
                    int delta = bit1.query(idx);
                    bit0.update(idx, -1);

                    ans += delta;
                } else {
                    int idx = list1.get(i/2);
                    int delta = bit0.query(idx);
                    bit1.update(idx, -1);
                    ans += delta;
                }
            }

        } else {

            if (list1.size() < list0.size()) return Long.MAX_VALUE;

            for (int i = 0; i < n; i++) {
                if (i % 2 == 0) {
                    int idx = list1.get(i/2);
                    int delta = bit0.query(idx);
                    bit1.update(idx, -1);

                    ans += delta;
                } else {
                    int idx = list0.get(i/2);
                    int delta = bit1.query(idx);
                    bit0.update(idx, -1);
                    ans += delta;
                }
            }
        }

        return ans;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] buf = sc.next().toCharArray();

        long res = Math.min(solve(buf.clone(), '0'), solve(buf.clone(), '1'));

        System.out.println(res);
    }

}

```

不过这个解法，应该是写复杂了。

直接贪心即可

就是${\sum_{i=0}^{i=(n+1)/2} (current_i - expect_i)}$

$current_i是第i个1所在的位子，expect_i是被期望的位子$

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {

    // *)
    public static long solve(char[] buf, char s) {
        // 找到最近的0
        int n = buf.length;
        
        List<Integer> list0 = new ArrayList<>();
        List<Integer> list1 = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (buf[i] == '0') {
                list0.add(i);
            } else {
                list1.add(i);
            }
        }

        long ans = 0l;

        if (s == '0') {
            if (list0.size() < list1.size()) return Long.MAX_VALUE;
            for (int i = 0; i < list0.size(); i++) {
                ans += Math.abs(2 * i - list0.get(i)); 
            }
        } else {
            if (list1.size() < list0.size()) return Long.MAX_VALUE;
            for (int i = 0; i < list1.size(); i++) {
                ans += Math.abs(2 * i - list1.get(i)); 
            }
        }

        return ans;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] buf = sc.next().toCharArray();

        long res = Math.min(solve(buf.clone(), '0'), solve(buf.clone(), '1'));

        System.out.println(res);
    }

}
```

---
## [D. 游游的9的倍数](https://ac.nowcoder.com/acm/contest/60245/D)

经典的同余DP

因为求的是9的倍数，而数字串前缀 $S_i = a_0a_1...a_i = b_0 * 9^x + b_1 * 9^{x-1}... + b_i * 9^0$

那$S_ia_{i+1} = 10 * b_0 * 9^x + 10 * b_1 * 9^{x-1}... + 10 * b_i * 9^0 + a_{i+1} = 10 * b_i + a_{i+1}$

所以我们令

$opt[i][j], 前i元素中，构建对9余数为j的总个数$

状态转移为

${opt[i+1][(j*10+a_{i+1}) \% 9] = opt[i][(j*10+a_{i+1})\%9] + opt[i][j]}$

当然这边对空集合的处理，有个小技巧，就是单独加1

$opt[i+1][a_{i+1}\%9] += 1$

可以借助滚动数组来优化DP的空间

复杂度分析，时间$O(9*n)$, 空间复杂度为$O(20)$

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] buf = sc.next().toCharArray();

        long mod = 10_0000_0007l;
        long[][] opt = new long[2][9];
        opt[0][0] = 0l;

        int prev = 0, next = 1;
        for (int i = 0; i < buf.length; i++) {
            int p = buf[i] - '0';

            // 添加/不添加
            Arrays.fill(opt[next], 0);

            for (int j = 0; j < 9; j++) {
                opt[next][j] = opt[prev][j];
            }
            for (int j = 0; j < 9; j++) {
                opt[next][(j * 10 + p) % 9] += opt[prev][j];
                opt[next][(j * 10 + p) % 9] %= mod;
            }

            opt[next][p % 9] = (opt[next][p % 9] + 1) % mod;

            prev = 1 - prev;
            next = 1 - next;
        }

        System.out.println(((opt[prev][0]) % mod + mod) % mod);

    }


}
```

如果9是个变量k，那其实也是同一种套路。

如果此题不用DP，能否解呢? 感觉有，因为9做为除数比较特殊，它打破了字符串的顺序限制。

---

# 写在最后

没有绝对的善意，关切的目光中也会掺杂恶意的期待。

![alt](./images/1h.png)