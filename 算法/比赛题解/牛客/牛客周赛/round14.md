# 牛客周赛 Round 14 题解报告 | 珂学家 | 环形模拟 + 滑窗&前缀和&二分 + 数学

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231008/446702330_1696776985790/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

这场牛客周赛很特别，没有一题是水题（签到），也没真正意义上的压轴题。

牛客周赛还是注重数学，B，D是数学题，A是模拟题，C的解法偏多，滑窗/26组前缀和&二分皆可。

---

## [A. 小红的环形字符串](https://ac.nowcoder.com/acm/contest/65821/A)

如果没有环形，那这题单纯使用一个 stack 就可以解决，就是括号匹配问题一样

那如果是环形结构呢？

其实这题有一个性质

x...AAA...y

中间的3个A，如果第一个和第二个匹配，还是第二个和第三个匹配，都不会影响结果

同理在环形链接处，也满足相似的结论。

---

核心结论:

> 根据等价原理，合并操作的顺序，并不影响最终结果

可以引入双端队列

先把双端队列当栈使用

最后比较双端队列的头和尾，进行抵消

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        char[] str = sc.next().toCharArray();
        
        // 等价原理
        // 1. 先当stack使用
        int ans = 0;
        Deque<Character> deq = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            char c = str[i];
            if (!deq.isEmpty() && deq.peekLast() == c) {
                ans += 2;
                deq.pollLast();
            } else {
                deq.addLast(c);
            }
        }

        // 2. 双端队列，合并头和尾
        while (deq.size() >= 2 && deq.peekFirst() == deq.peekLast().charValue()) {
            ans += 2;
            deq.pollFirst();
            deq.pollLast();
        }

        System.out.println(ans);
    }

}
```

---
## [B. 小红的乘除变换](https://ac.nowcoder.com/acm/contest/65821/B)

这题的5和6其实很特别互质

可以从因子5,6的角度切入

x的5,6的因子个数nx5, nx6

y的5,6的因子个数ny5, ny6

令tx为x去除5,6因子后的数，ty为y去除5,6因子后的数

> 如果  tx != ty, 则 x 不可能转化为 y

> 5是乘，6是除，那 nx5 > ny5 || nx6 < ny6, 也不存在解

> 最后的操作数为 ny5 - nx5 + nx6 - ny6

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int t = sc.nextInt();
        while (t-- > 0) {
            long x = sc.nextLong();
            long y = sc.nextLong();

            int ny5 = 0, ny6 = 0;
            while (y > 0 && y % 5 == 0) { y /= 5; ny5++; }
            while (y > 0 && y % 6 == 0) { y /= 6; ny6++; }

            int nx5 = 0, nx6 = 0;
            while (x > 0 && x % 5 == 0) { x /= 5; nx5++; }
            while (x > 0 && x % 6 == 0) { x /= 6; nx6++; }

            if (x != y || nx5 > ny5 || nx6 < ny6) {
                System.out.println(-1);
            } else {
                System.out.println(ny5 - nx5 + nx6 - ny6);
            }
        }
    }

}

```

---

## [C. 小红的子串](https://ac.nowcoder.com/acm/contest/65821/C)

按照定义，连续区间 [a, b]其不同的字符数 在 [l, r]之间，被小红心中的意中人。

求这样的子区间总个数 ?

因为字符串的长度为 $2\times 10^5$, 因此没法从枚举左右两端点入手。

大致有两种思路

- 滑窗
- 枚举左(右)端点, 然后二分求右(左)边界

---
### 滑窗

滑窗可以用三指针，也可以用双指针

三指针，枚举右(左)端点, 然后滑窗维护[l, r]不同字符种类数的边界p1, p2, 结果为

ans += abs(p2 - p1)

双指针，定义f(x)为种类数 >= x的方案总数

res = f(l) - f(r + 1)


```java [group1-三指针]
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int l = sc.nextInt(), r = sc.nextInt();

        char[] str = sc.next().toCharArray();

        Map<Integer, Integer> hash1 = new HashMap<>();
        Map<Integer, Integer> hash2 = new HashMap<>();

        long ans = 0;
        int p1 = 0, p2 = 0;
        for (int i = 0; i < n; i++) {
            int v = str[i] - 'a';
            hash1.merge(v, 1, Integer::sum);
            hash2.merge(v, 1, Integer::sum);

            // 维护有r种不同字符的边界, > r
            while (p1 <= i && hash1.size() > r) {
                hash1.computeIfPresent(str[p1] - 'a', (a, b) -> b > 1 ? b - 1 : null);
                p1++;
            }
            // 维护有l种不同字符的边界, >=l
            while (p2 <= i && hash2.size() >= l) {
                hash2.computeIfPresent(str[p2] - 'a', (a, b) -> b > 1 ? b - 1 : null);
                p2++;
            }

            ans += (p2 - p1);
        }

        System.out.println(ans);
    }

}
```

```java [group1-双指针]
import java.io.*;
import java.util.*;

public class Main {
    
  
    /*
        双指针，定义f(x)为种类数 >= x的方案总数
        res = f(l) - f(r + 1) 
    */
    static long solve(char[] str, int x) {
        Map<Integer, Integer> hash = new HashMap<>();

        long ans = 0;
        int j = 0;
        for (int i = 0; i < str.length; i++) {
            int v = str[i] - 'a';
            hash.merge(v, 1, Integer::sum);
            while (j <= i && hash.size() >= x) {
                hash.computeIfPresent(str[j] - 'a', (a, b) -> b > 1 ? b - 1 : null);
                j++;
            }
            ans += j;
        }
        return ans;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int l = sc.nextInt(), r = sc.nextInt();

        char[] str = sc.next().toCharArray();
        System.out.println(solve(str, l) - solve(str, r + 1));
    }

}
```

---

### 前缀和&二分

字符种类最多是26，是有限可枚举

因此这也是一个切入点

构建26 slots的前缀和

然后求二分求 [l, r]的两个边界


```java
import java.io.*;
import java.util.*;

public class Main {

    static int binarySearch(int[][] pre, int e, int x) {
        int l = 0, r = e;
        while (l <= r) {
            int m = l + (r - l) / 2;

            // 统计不同字符种类个数
            int cnt = 0;
            for (int i = 0; i < 26; i++) {
                if (pre[i][e + 1] - pre[i][m] > 0) cnt++;
            }
            // cnt >= x 是 check核心逻辑
            if (cnt >= x) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        return r;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int l = sc.nextInt(), r = sc.nextInt();

        char[] str = sc.next().toCharArray();

        int[][] pre = new int[26][n + 1];

        // 构建26个前缀和
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < 26; j++) {
                pre[j][i + 1] = pre[j][i] + (str[i] - 'a' == j ? 1 : 0);
            }
        }

        long ans = 0;
        for (int i = 0; i < n; i++) {
            ans += binarySearch(pre, i, l) - binarySearch(pre, i, r + 1);
        }

        System.out.println(ans);
    }

}

```

---

## [D. 回文权值和](https://ac.nowcoder.com/acm/contest/65821/D)

算一道高中数学题吧

就是求$26^n$的串中，连续长度为3的回文子串总数有多少个

假设(i, i+1, i+2)，其构成回文的种类为$26\times26=676$个，那在$26^n$种可能字符串中。

一共出现为 $26^{n-3}$次

同时，这样的三元组，在n长的字符串中，一共可以枚举$n - 2$

综合得

$$ res = 676 \times (n-2) \times 26^{n-3}$$

> 676 * (n - 2) * 26^(n-3)

考察了快速幂这个知识点

```java
import java.io.*;
import java.util.*;

public class Main {

    public static long ksm(long b, long v, long mod) {
        long r = 1l;
        b = b % mod;
        while (v > 0) {
            if (v % 2 == 1) r = r * b % mod;
            b = b * b % mod;
            v /= 2;
        }
        return r;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long n = sc.nextLong();
        // n < 3没有解
        if (n < 3) {
            System.out.println(0);
            return;
        }

        long mod = (long)1e9 + 7;
        long res = 676l * ksm(26, n - 3, mod) % mod * ((n - 2) % mod) % mod;
        System.out.println(res);
    }

}
```

---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231008/446702330_1696777017884/D2B5CA33BD970F64A6301FA75AE2EB22)
