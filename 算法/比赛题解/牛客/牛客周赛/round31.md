
# 牛客周赛 Round 31 解题报告 | 珂学家 | 设计 + 组合

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20240204/446702330_1707058787514/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

D题出的蛮好的，其实做过LruCache题的同学，基本都会，即Map+双向链表技巧。

E题典型的DP题，负数可以引入偏移来解决。

F题是道数学题，组合+乘法原理。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)


---

## [A. 小红小紫替换](https://ac.nowcoder.com/acm/contest/74362/A)

思路: 模拟

```python3 []
s = input()

if s == "kou":
    print ("yukari")
else:
    print (s)
```

---
## [B. 小红的因子数](https://ac.nowcoder.com/acm/contest/74362/B)

思路: 质因子拆解

```python3 []
x = int(input())

def factors(n: int) -> int:
    res = 0
    i = 2
    while i <= n // i:
        if n % i == 0:
            res += 1
            while n % i == 0:
                n = n // i
        i += 1
    if n > 1:
        res += 1
    return res
    
print (factors(x))
```

---
## [C. 小红的字符串中值](https://ac.nowcoder.com/acm/contest/74362/C)

思路: 枚举

对于i个字符，如果其为目标字符

则可提供的奇数子字符串数位：

min(i+1,n - i)

```python3 []
nn, t = input().split()
n = int(nn)
s = input()

res = 0
for i in range(n):
    if s[i] == t[0]:
        d = min(i, n - 1 - i) + 1
        res += d 

print (res)

```

---
## [D. 小红数组操作](https://ac.nowcoder.com/acm/contest/74362/D)

思路: 复合数据结构

- map

- 双向链表

双向链表，方便数据的插入和删除， map方便数据的快速定位

很典的题

```java []
import java.io.BufferedInputStream;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    static
    class Structure {
        static
        class Node {
            public Node left, right;
            int val;
            public Node() {}
            public Node(int val) {
                this.val = val;
            }
        }

        Map<Integer, Node> hash = new HashMap<>();
        Node head, tail;

        public Structure() {
            head = new Node();
            tail = new Node();
            head.right = tail;
            tail.left = head;
        }

        void add(int x, int y) {
            Node yn = hash.get(y);
            Node cur = new Node(x);
            Node second = yn.right;
            yn.right = cur;
            cur.left = yn;
            cur.right = second;
            second.left = cur;
            hash.put(x, cur);
        }

        void addHead(int x) {
            Node cur = new Node(x);
            Node second = head.right;
            head.right = cur;
            cur.left = head;
            cur.right = second;
            second.left = cur;
            hash.put(x, cur);
        }

        void remove(int x) {
            Node xn = hash.get(x);
            Node left = xn.left, right = xn.right;
            left.right = right;
            right.left = left;
            hash.remove(x);
        }

        List<Integer> toList() {
            List<Integer> res = new ArrayList<>();
            Node iter = head.right;
            while (iter != tail) {
                res.add(iter.val);
                iter = iter.right;
            }
            return res;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        Structure structure = new Structure();
        int n = sc.nextInt();
        for (int i = 0; i < n; i++) {
            int op = sc.nextInt();
            if (op == 1) {
                int x = sc.nextInt(), y = sc.nextInt();
                if (y == 0) {
                    structure.addHead(x);
                } else {
                    structure.add(x, y);
                }
            } else {
                int x = sc.nextInt();
                structure.remove(x);
            }
        }
        List<Integer> res = structure.toList();

        System.out.println(res.size());
        System.out.println(res.stream().map(String::valueOf).collect(Collectors.joining(  " ")));
    }

}
```

---
## [E. 小红的子集取反](https://ac.nowcoder.com/acm/contest/74362/E)

思路: 动态规划

### 1. 线性DP

构建dp[i][j], i为前i项，j为构建和，其值为最小的取反次数

状态转移

dp[i][j] = min(dp[i - 1][j + arr[i]], dp[i - 1][j - arr[j]] + 1)

因为是线性，所以借助滚动数组进行优化

```java []
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        int inf = Integer.MAX_VALUE / 10;
        int offset = n * 200;
        int[] dp = new int[2 * offset + 1];
        Arrays.fill(dp, inf);

        dp[offset] = 0;

        for (int i = 0; i < n; i++) {
            int[] dp2 = new int[2 * offset + 1];
            Arrays.fill(dp2, inf);
            for (int j = 0; j <= 2 * offset; j++) {
                if (dp[j] == inf) continue;
                if (j + arr[i] <= 2 * offset && j + arr[i] >= 0) {
                    if (dp2[j + arr[i]] > dp[j]) {
                        dp2[j + arr[i]] = dp[j];
                    }
                }
                if (j - arr[i] >= 0 && j - arr[i] <= 2 * offset) {
                    if (dp2[j - arr[i]] > dp[j] + 1) {
                        dp2[j - arr[i]] = dp[j] + 1;
                    }
                }
            }
            dp = dp2;
        }

        if (dp[offset] == inf) {
            System.out.println(-1);
        } else {
            System.out.println(dp[offset]);
        }
    }

}
```

### 2. 0-1背包DP

除了这个思路外，也可以引入方程组

- x + y = s

- x - y = 0

推导 y = s / 2, s必然为偶数

问题就变为经典的0-1背包问题，从集合中挑选最小的元素，使得其为S/2.

```java []
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int sum = 0;
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            sum += arr[i];
        }

        if (sum % 2 == 1) {
            System.out.println(-1);
            return;
        }

        int inf = Integer.MAX_VALUE / 10;
        int offset = n * 200; // 值域范围
        int[] dp = new int[2 * offset + 1];
        Arrays.fill(dp, inf);

        dp[offset] = 0;
        for (int i = 0; i < n; i++) {
            int[] dp2 = dp.clone();
            // 逻辑变成取或不取
            for (int j = 0; j < dp.length; j++) {
                if (j + arr[i] >= 0 && j + arr[i] <= 2 * offset) {
                    if (dp2[j + arr[i]] > dp[j] + 1) {
                        dp2[j + arr[i]] = dp[j] + 1;
                    }
                }
            }
            dp = dp2;
        }

        if (dp[offset + sum/2] == inf) {
            System.out.println(-1);
        } else {
            System.out.println(dp[offset + sum/2]);
        }
    }

}
```


---
## [F. 小红的连续段](https://ac.nowcoder.com/acm/contest/74362/F)

思路: 组合+乘法原理

对于t个段，其由n1 = t/2, n2 = (t+1)/2 两部分组成

根据隔板法

令a字母构建n1端，其组合数为 C(x - 1, n1 - 1)

令b字母构建n2端，其组合数为 C(y - 1, n2 - 1)

综合为 C(x-1, n1-1) * C(y-1, n2-1)

而a为n2段，b为n1段，依然成立

```java []
import java.io.*;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    static class Factorial {
        int n;
        long mod;
        long[] fac, inv;

        public Factorial(int n, long mod) {
            this.n = n;
            this.mod = mod;
            this.fac = new long[n + 1];
            this.inv = new long[n + 1];

            fac[0] = 1;
            for (int i = 1; i <= n; i++) {
                fac[i] = fac[i - 1] * i % mod;
            }
            // 费马小定律
            inv[n] = ksm(fac[n], mod - 2);
            for (int i = n - 1; i >= 0; i--) {
                inv[i] = inv[i + 1] * (i + 1) % mod;
            }
        }

        public long fac(int m) {
            return fac[m];
        }

        public long facInv(int m) {
            return inv[m];
        }

        public long inv(int m) {
            // 0没有逆元
            return inv[m] * fac[m - 1] % mod;
        }

        public long comb(int m, int k) {
            return fac[m] * inv[k] % mod * inv[m - k] % mod;
        }

        private long ksm(long b, long v) {
            long r = 1l;
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

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int x = sc.nextInt(), y = sc.nextInt();

        int n = x + y;
        long mod = (long)1e9 + 7;
        Factorial factorial = new Factorial(n, mod);

        List<Long> ans = new ArrayList<>();
        for (int i = 1; i <= n; i++) {
            long res = 0;
            int a = (i + 1) / 2;
            int b = i / 2;
            if (a <= x && a >= 1 && b <= y && b >= 1) {
                res += factorial.comb(x - 1, a - 1) * factorial.comb(y - 1, b - 1) % mod;
                res %= mod;
            }
            if (a <= y && a >= 1 && b <= x && b >= 1) {
                res += factorial.comb(y - 1, a - 1) * factorial.comb(x - 1, b - 1) % mod;
                res %= mod;
            }
            ans.add(res);
        }

        System.out.println(ans.stream().map(String::valueOf).collect(Collectors.joining("\n")));
    }

}
```

---
# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20240204/446702330_1707058828209/D2B5CA33BD970F64A6301FA75AE2EB22)



