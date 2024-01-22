
# 牛客周赛 Round 29 解题报告 | 珂学家 | 博弈&概率DP

# 前言


![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240122/446702330_1705853696692/D2B5CA33BD970F64A6301FA75AE2EB22)

--- 
# 整体评价

F题真心好题，很典，学到了很多。D题用了对顶堆，写到一半就想到了更简单的方法，哭。E题是基于众数的构造。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红大战小紫](https://ac.nowcoder.com/acm/contest/73422/A)

思路: 模拟

```python []
n, m = list(map(int, input().split()))

if n > m:
    print("kou")
elif n < m:
    print("yukari")
else:
    print("draw")
```

---

## [B. 小红的白日梦](https://ac.nowcoder.com/acm/contest/73422/B)

思路: 模拟

```python []
n = int(input())
day = input()
night = input()

res = 0
for i in range(n):
    if day[i] == 'Y' and night[i] == 'Y':
        res += 3
    elif day[i] == 'Y' and night[i] == 'N':
        res += 2
    elif day[i] == 'N' and night[i] == 'Y':
        res += 2

print(res)
```

---

## [C. 小红的小小红](https://ac.nowcoder.com/acm/contest/73422/C)

思路: 贪心构造

先构造xiaohong, 后续字符按剩余数量追加即可

```ptyhon[] 
from collections import Counter

s = input()

cnt = Counter(s)

res = []
for c in "xiaohong":
    res.append(c)
    cnt[c] -= 1
    
for (k, v) in cnt.items():
    if v > 0:
        res.extend([k] * v)
    
print(''.join(res))
```

---

## [D. 小红的中位数](https://ac.nowcoder.com/acm/contest/73422/D)

思路: 对顶堆

写复杂了，中间状态维护太麻烦了。

仅供负面教材典型。


```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.PriorityQueue;
import java.util.StringTokenizer;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        // 对顶堆
        TreeMap<Integer, Integer> h1 = new TreeMap<>();
        TreeMap<Integer, Integer> h2 = new TreeMap<>();
        int cnt1 = 0, cnt2 = 0;

        for (int i = 0; i < n; i++) {
            if (cnt1 == cnt2) {
                h1.merge(arr[i], 1, Integer::sum);
                cnt1++;
            } else {
                h2.merge(arr[i], 1, Integer::sum);
                cnt2++;
            }

            if (cnt1 > 0 && cnt2 > 0 && h1.lastKey() > h2.firstKey()) {
                int k1 = h1.lastKey();
                int k2 = h2.firstKey();

                h1.computeIfPresent(k1, (a, b) -> b > 1 ? b - 1 : null);
                h2.computeIfPresent(k2, (a, b) -> b > 1 ? b - 1 : null);

                h1.merge(k2, 1, Integer::sum);
                h2.merge(k1, 1, Integer::sum);
            }
        }

        double[] f = new double[n];
        for (int i = 0; i < n; i++) {
            if (h1.containsKey(arr[i])) {
                h1.computeIfPresent(arr[i], (a, b)-> b > 1 ? b - 1 : null);
                cnt1--;
            } else {
                h2.computeIfPresent(arr[i], (a, b)-> b > 1 ? b - 1 : null);
                cnt2--;
            }
            if (cnt1 < cnt2) {
                int k2 = h2.firstKey();
                h2.computeIfPresent(k2, (a, b) -> b > 1 ? b - 1 : null);
                h1.merge(k2, 1, Integer::sum);
                cnt1++;
                cnt2--;
            } else if (cnt1 > cnt2 + 1) {
                int k1 = h1.lastKey();
                h1.computeIfPresent(k1, (a, b) -> b > 1 ? b - 1 : null);
                h2.merge(k1, 1, Integer::sum);
                cnt1--;
                cnt2++;
            }

            if (cnt1 > cnt2) {
                f[i] = h1.lastKey();
            } else {
                f[i] = (h1.lastKey() + h2.firstKey()) / 2.0;
            }

            if (cnt1 > cnt2) {
                h2.merge(arr[i], 1, Integer::sum);
                cnt2++;
            } else {
                h1.merge(arr[i], 1, Integer::sum);
                cnt1++;
            }

            while (cnt1 > 0 && cnt2 > 0 && h1.lastKey() > h2.firstKey()) {
                int k1 = h1.lastKey();
                int k2 = h2.firstKey();

                h1.computeIfPresent(k1, (a, b) -> b > 1 ? b - 1 : null);
                h2.computeIfPresent(k2, (a, b) -> b > 1 ? b - 1 : null);

                h1.merge(k2, 1, Integer::sum);
                h2.merge(k1, 1, Integer::sum);
            }
        }
        System.out.println(Arrays.stream(f).mapToObj(x -> String.format("%.1f", x)).collect(Collectors.joining("\n")));
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

## [E. 小红构造数组](https://ac.nowcoder.com/acm/contest/73422/E)

思路: 众数+构造

只要 众数 <=(m+1)/2, m为数组长度, 则一定可以构造

按频率(从大到小)优先填充偶数位(0-index)，然后在填充奇数位，这样天然保证相同元素被隔离

可以反证，证明该思路，一定是OK的


```java []

import java.io.*;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    static List<long[]> split(long n) {
        List<long[]> res = new ArrayList<>();
        for (long i = 2; i <= n / i; i++) {
            if (n % i == 0) {
                int cnt = 0;
                while (n % i == 0) {
                    n /= i;
                    cnt++;
                }
                res.add(new long[] {i, cnt});
            }
        }
        if (n > 1) {
            res.add(new long[] {n, 1});
        }
        return res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long e = sc.nextLong();
        if (e == 1) {
            System.out.println(-1);
            return;
        }
        
        List<long[]> primes = split(e);

        // 感觉像众数的构造
        int n = 0;
        Collections.sort(primes, Comparator.comparing(x -> -x[1]));
        int m = (int)primes.get(0)[1];
        for (long[] x: primes) {
            n += (int)x[1];
        }
        if (m > (n + 1) / 2) {
            System.out.println(-1);
        } else {
            Long[] res = new Long[n];
            int ptr = 0;
            for (long[] x: primes) {
                for (int j = 0; j < x[1]; j++) {
                    res[ptr] = x[0];
                    ptr += 2;
                    if (ptr >= n) ptr = 1;
                }
            }
            System.out.println(n);
            System.out.println(Arrays.stream(res).map(String::valueOf).collect(Collectors.joining(" ")));
        }
    }

}
```

---

## [F. 小红又战小紫](https://ac.nowcoder.com/acm/contest/73422/F)

背景: 博弈+概率DP

思路: 状态压缩 + 博弈简化

---

因为和堆的位子无关，只和堆的石头数有关。

因此可以把很多堆(只有1和2)，压缩为(1个石头的堆数，2个石头的堆数)

这样在博弈过程中，其状态就从n维(n<=1000)降维压缩到2维

---

博弈简化

博弈是套着 alpha+beta剪枝

但是这里面其实有策略挑选

- 如果选用1技能，得到胜率概率为p1
- 如果选用2技能，得到胜率概率为p2

取其 max(p1, p2)

因为概率取模会丢失大小关系，所以这边需要保留真实的概率值，这样就很麻烦。

---

这边有个特殊情况

就是当1,2个数得石头堆都存在时，如果选用技能2，那一定输，也就是胜率p2=0

在这个前提下

博弈策略，只需要单独评估技能1就行(除只有1个石头堆的情况)，这样就可以runtime过程中，使用模。

---

- 记忆化搜索

dp[i][j] = j / inv(i + j) * (1 - dp[i+1][j-1]) + i / inv(i+j) * (1 - dp[i - 1][j])

---

- 题外话

如果这题没有限制石头数目为1~2之间，那么这题得保留真实的概率，而不是模意义下的概率。

---


```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;
import java.util.StringTokenizer;

public class Main {

    static long mod = (long)1e9 + 7;

    static int MZ = 1000;
    static long[] gFac = new long[MZ + 1];
    static long[] gInv = new long[MZ + 1];
    static {
        gFac[0] = 1;
        for (int i = 1; i <= MZ; i++) {
            gFac[i] = gFac[i-1] * i % mod;
        }
        gInv[MZ] = BigInteger.valueOf(gFac[MZ]).modInverse(BigInteger.valueOf(mod)).longValue();
        for (int i = MZ - 1; i >= 0; i--) {
            gInv[i] = gInv[i + 1] * (i + 1) % mod;
        }
    }

    static Map<Long, Long> memo = new HashMap<>();
    static long dfs(int x, int y) {
        // 直接使用技能2
        if (x > 0 && y == 0) {
            return 1;
        }
        // 必输态
        if (x == 0 && y == 0) {
            return 0;
        }

        long k = ((long)x << 32) | y;
        if (memo.containsKey(k)) {
            return memo.get(k);
        }

        long inv = gInv[x + y] * gFac[x + y - 1] % mod;

        // 策略1，随机堆
        long r = 0;
        if (x > 0) {
            long p = dfs(x - 1, y);
            r += (1 - p) * x % mod * inv % mod;
        }
        if (y > 0) {
            long p = dfs(x + 1, y - 1);
            r += (1 - p) * y % mod * inv % mod;
        }

        r = (r % mod + mod) % mod;
        memo.put(k, r);
        return r;
    }


    public static void main(String[] args) {
        AReader sc = new AReader();
        int x = 0, y = 0;
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            if (arr[i] == 1) x++;
            else if (arr[i] == 2) y++;
        }
        long p = dfs(x, y);
        System.out.println(p);
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

# 写在最后
![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240122/446702330_1705853292414/D2B5CA33BD970F64A6301FA75AE2EB22)


