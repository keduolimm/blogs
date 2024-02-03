
# 牛客周赛 Round 30 解题报告 | 珂学家 | 树形DP + 期望DP

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20240128/446702330_1706449942267/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

D是一道数学题，E是一道经典的入门树形DP，F题是一道期望DP，记忆化的方式更加简单一些。

ABC虽然偏简单，但是都是构造形态的，好像有CF风格了。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的删字符](https://ac.nowcoder.com/acm/contest/73760/A)

思路: 模拟

题意指定长度为3

```python3 []
s = input()
print (s[0] + s[2])
```

---

## [B. 小红的正整数](https://ac.nowcoder.com/acm/contest/73760/B)

思路: 贪心 + 构造

先选择一个非零的数，然后按自然序构造即可

```python3 []
from collections import Counter

x = input()
cnt = Counter(x)

res = []
for i in range(1, 10):
    k = chr(i + ord('0'))
    if cnt[k] > 0:
        res.append(k)
        cnt[k] -= 1
        break
        
for i in range(0, 10):
    k = chr(i + ord('0'))
    # repeat k
    res.extend(k * cnt[k])
        
print (''.join(res))
```


---

## [C. 小红构造回文](https://ac.nowcoder.com/acm/contest/73760/C)

思路: 寻找不同的字符

限定在前半段(不包含奇数长度的中心节点)

然后交换即可

```python3 []
s = input()

s = [c for c in s]
n = len(s)

p = -1
for i in range(1, n//2):
    if s[0] != s[i]:
        p = i
        break

# not found        
if p == -1:
    print (-1)
else:
    # swap
    s[0], s[p] = s[p], s[0]
    s[n - 1], s[n - 1 - p] = s[n - 1 - p], s[n - 1]
    print (''.join(s))
```

---

## [D. 小红整数操作](https://ac.nowcoder.com/acm/contest/73760/D)

思路: 数学

先求a,b的最小形态，即皆除以最大公约数gcd，得到

c = a/gcd(a,b)

d = b/gcd(a,b)

假定$c \le d$

然后就是数学中的范围收敛知识了

最小倍数l = (x + c - 1) / c

最大倍数r = y / d

然后求[l, r]的区间个数，即为答案


```java []
import java.io.*;
import java.util.*;

public class Main {

    static int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int a = sc.nextInt(), b = sc.nextInt();
        int x = sc.nextInt(), y = sc.nextInt();

        // swap
        if (a > b) {
            int t = a;
            a = b;
            b = t;
        }

        int g = gcd(a, b);
        int c = a/g;
        int d = b/g;

        // 范围组合
        int d1 = (x + c - 1) / c;
        int d2 = y / d;

        if (d1 <= d2) {
            System.out.println(d2 - d1 + 1);
        } else {
            System.out.println(0);
        }
    }

}
```

---

## [E. 小红树上染色](https://ac.nowcoder.com/acm/contest/73760/E)

思路: 树形DP

每个节点有两种状态, 为白色/红色

因为限制是不能存在两个相邻节点为白色

所以如果当前节点为白色，其子节点必须都是红色节点

如果当前节点为红色，其子节点红白即可

状态叠加是基于乘法原理的

$white[u] = \prod_{v\in u的子节点} red[v]$

$red[u] = \prod_{v\in u的子节点} (red[v] \oplus white[v])$

注: 这边是+号

```java []
import java.io.*;
import java.util.*;

public class Main {

    static long mod = (long)1e9 + 7;

    static class Solution {
        int n;
        List<Integer> []g;
        long[] white;
        long[] red;

        long solve(int n, List<Integer> []g) {
            this.n = n;
            this.g = g;

            this.white = new long[n];
            this.red = new long[n];
            dfs(0, -1);
            return (white[0] + red[0]) % mod;
        }

        void dfs(int u, int fa) {
            long w = 1, r = 1;
            for (int v: g[u]) {
                if (v == fa) continue;
                dfs(v, u);

                w = w * red[v] % mod;
                r = r * ((white[v] + red[v]) % mod) % mod;
            }
            white[u] = w;
            red[u] = r;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        // 树形DP
        int n = sc.nextInt();
        List<Integer>[]g = new List[n];
        Arrays.setAll(g, x->new ArrayList<>());
        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt() - 1 , v = sc.nextInt() - 1;
            g[u].add(v);
            g[v].add(u);
        }
        Solution solution = new Solution();
        long r = solution.solve(n, g);
        System.out.println(r);
    }

}
```

---

## [F. 小红叒战小紫](https://ac.nowcoder.com/acm/contest/73760/F)

思路: 期望DP

构建4维DP，记忆化搜索

关键在于

- 状态压缩
- 退出条件
- 期望E的推导公式

```java []
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {

    static long mod = (long)1e9 + 7;

    static class Solution {

        Long[][][][] opt;

        long solve(int n1, int n2, int m1, int m2) {
            opt = new Long[n1 + 1][n2 + 1][m1 + 1][m2 + 1];
            return dfs(n1, n2, m1, m2);
        }

        long tx(long a) {
            return (a % mod + mod) % mod;
        }

        long dfs(int s1, int s2, int e1, int e2) {
            if (s1 == -1 || e1 == -1) return tx(-1);

            if (s1 + s2 == 0) return 0;
            if (e1 + e2 == 0) return 0;
            if (s1 == 0 && e1 == 0) return 0;
            if (s2 == 0 && e2 == 0) return 0;
            if (s1 == 0 && s2 > 0 && e2 == 0) return e1;
            if (e2 > 0 && e1 == 0 && s2 == 0) return s1;

            if (opt[s1][s2][e1][e2] != null) return opt[s1][s2][e1][e2];

            int p1 = s1 + s2;
            int p2 = e1 + e2;

            long r1 = dfs(s1 - 1, s2, e1, e2);
            r1 = tx(r1);
            r1 = r1 * s1 % mod * inv(p1) % mod * e2 % mod * inv(p2) % mod;

            long r2 = dfs(s1, s2, e1 - 1, e2);
            r2 = tx(r2);
            r2 = r2 * s2 % mod * inv(p1) % mod * e1 % mod * inv(p2) % mod;

            long r = (r1 + r2 + 1) % mod;
            r = tx(r);

            r = r * p1 % mod * p2 % mod * inv(tx(tx(tx(p1 * p2) - tx(s1 * e1)) - tx(s2 * e2))) % mod;

            return opt[s1][s2][e1][e2] = r;
        }

        // 逆元构建
        Map<Long, Long> memo = new HashMap<>();
        long inv(long v) {
            if (memo.containsKey(v)) return memo.get(v);
            long r = BigInteger.valueOf(v).modInverse(BigInteger.valueOf(mod)).longValue();
            memo.put(v, r);
            return r;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), m = sc.nextInt();
        int n1 = 0, n2 = 0;
        int m1 = 0, m2 = 0;

        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            if (v == 1) n1++;
            else n2++;
        }
        for (int i = 0; i < m; i++) {
            int v = sc.nextInt();
            if (v == 1) m1++;
            else m2++;
        }

        Solution solution = new Solution();
        long r = solution.solve(n1, n2, m1, m2);
        System.out.println(r);
    }

}
```



---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20240128/446702330_1706450011609/D2B5CA33BD970F64A6301FA75AE2EB22)

