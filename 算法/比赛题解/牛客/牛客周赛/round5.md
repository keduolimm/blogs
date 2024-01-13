# 牛客周赛 Round 5 解题报告 | 珂学家 | 思维场


---

# 前言



![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230730/446702330_1690722054226/D2B5CA33BD970F64A6301FA75AE2EB22)


剑，和茶一样，只有细细品味，才能理解它的风雅。

---

# 整体评价

挺难的一场比赛，C题差点点错科技树(想着用Dsu On Tree), D题开始上难度，但是只是分析其实就是一个区间求交集的脑筋急转弯，E题盲猜是菊花图。


---
## [A. 游游的字母变换](https://ac.nowcoder.com/acm/contest/62033/A)

模拟题吧，把大字字母变成下一位，小写字母前一位

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        for (int i = 0; i < str.length; i++) {
            char c = str[i];
            if (c >= 'A' && c <= 'Z') {
                str[i] = (char)(((c - 'A') + 1) % 26 + 'A');
            } else if (c >= 'a' && c <= 'z') {
                str[i] = (char)((c - 'a' + 25) % 26 + 'a');
            }
        }
        System.out.println(new String(str));
    }

}
```

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    string s;
    cin >> s;
    int n = s.length();
    for (int i = 0; i < n; i++) {
        char c = s[i];
        if (c >= 'a' && c <= 'z') {
            s[i] = (char)((c - 'a' + 25) % 26 + 'a');
        } else if (c >= 'A' && c <= 'Z') {
            s[i] = (char)((c - 'A' + 1) % 26 + 'A');
        }
    }
    cout << s << endl;
    
    return 0;
}
```

---

## [B. 游游的排列构造](https://ac.nowcoder.com/acm/contest/62033/B)

构造题，就是选择k个最大的数，从小到大顺序填充$2*i$的位置
然后剩下的n-k个数，从大到小填充剩下的位置

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), k = sc.nextInt();

        int[] res = new int[n];
        int ptr = n - k, mptr = n - k + 1;
        for (int i = 0; i < n; i++) {
            if (i <= 2 * k - 1) {
                if (i % 2 == 0) {
                    // [n-k+1, n], 这k个最大的数, 从小到大填充2*i的位置
                    res[i] = mptr++;
                } else {
                    // 剩下的数，从大到小填充剩余的位置
                    res[i] = ptr--;
                }
            } else {
                res[i] = ptr--;
            }
        }

        System.out.println(Arrays.stream(res)
                           .mapToObj(String::valueOf).collect(Collectors.joining(" ")));
    }

}
```


---

## [C. 游游的二进制树](https://ac.nowcoder.com/acm/contest/62033/C)

因为树的节点数n，只有$n\le1000$, 所以$O(n^2)$的时间复杂度可以接受

枚举每个顶点，然后进行BFS，统计在[l,r]的个数

```java

import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static class Solution {

        long solve(char[] str, List<Integer> []g, long l, long r) {

            int n = str.length;
            long ans = 0;
            for (int i = 1; i <= n; i++) {
                // 枚举起点
                ans += bfs(i, str, g, l, r);
            }
            return ans;
        }

        long bfs(int u, char[] str, List<Integer>[] g, long l, long r) {
            long ans = 0;
            boolean[] used = new boolean[str.length + 1];
            Deque<long[]> deq = new ArrayDeque<>();
            deq.offer(new long[] {u, str[u - 1] - '0'});
            used[u] = true;

            while (!deq.isEmpty()) {
                long[] cur = deq.poll();
                int v = (int)cur[0];
                for (int t: g[v]) {
                    if (used[t]) continue;
                    used[t] = true;
                    long vx = (cur[1] << 1) + (str[t - 1] - '0');
                    if (vx > r) continue;
                    if (vx >= l && vx <= r) ans++;
                    if (vx <= r) {
                        deq.offer(new long[] {t, vx});
                    }
                }
            }
            return ans;
        }

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        long l = sc.nextLong(), r = sc.nextLong();

        char[] str = sc.next().toCharArray();

        List<Integer>[] g = new List[n + 1];
        Arrays.setAll(g, x -> new ArrayList<>());
        for (int i = 0; i < n - 1; i++) {
            int u = sc.nextInt(), v = sc.nextInt();
            g[u].add(v);
            g[v].add(u);
        }

        Solution solution = new Solution();
        long res = solution.solve(str, g, l, r);

        System.out.println(res);

    }

}
```

---

## [D. 游游的矩阵统计](https://ac.nowcoder.com/acm/contest/62033/D)

这题蛮有意思，一开始想着，是否可以正难则反, 后来返现这题不是 $\gt 2种不同颜色$，是 “恰好”

那这题如何求解呢？

其实这题关键还是区间交集

可以观察到，如下满足需求
1. 上下两行中，两个不同颜色(color)区间的交集
2. 上下两行中，两个不同颜色(color)区间的交集的边界
3. 上下两行中，两个相同颜色(color)区间的交集的边界

大概就是这个思路，然后就是枚举双指针

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        long n = sc.nextLong();
        int m1 = sc.nextInt();
        int m2 = sc.nextInt();

        // 都不好做
        long[][] arr = new long[m1][2];
        long[][] brr = new long[m2][2];

        for (int i = 0; i < m1; i++) {
            arr[i][0] = sc.nextLong();
            arr[i][1] = sc.nextLong();
        }
        for (int i = 0; i < m2; i++) {
            brr[i][0] = sc.nextLong();
            brr[i][1] = sc.nextLong();
        }

        // 找不同点
        // 正难则反
        long ans = 0;
        int idx1 = 0, idx2 = 0;

        long cnt1 = 0, cnt2 = 0;
        while (idx1 < arr.length && idx2 < brr.length) {
            long l1 = cnt1, r1 = cnt1 + arr[idx1][1] - 1;
            long l2 = cnt2, r2 = cnt2 + brr[idx2][1] - 1;


            long nl = Math.max(l1, l2);
            long nr = Math.min(r1, r2);

            // 这边进行计算
            if (arr[idx1][0] == brr[idx2][0]) {
                if (l1 < nl) {
                    ans++;
                } else if (l2 < nl) {
                    ans++;
                } else if (l1 == nl && l2 == nl && idx1 > 0 && idx2 > 0 && arr[idx1 - 1][0] == brr[idx2 - 1][0]) {
                    ans++;
                }
            } else {
                if (nl <= nr) {
                    ans += (nr - nl);
                }

                if (l1 < nl && idx2 > 0 && (brr[idx2 - 1][0] == arr[idx1][0])) {
                    ans++;
                } else if (l2 < nl && idx1 > 0 && (arr[idx1 - 1][0] == brr[idx2][0])) {
                    ans++;
                } else if (l1 == nl && l2 == nl) {
                    if (idx1 > 0 && idx2 > 0 && arr[idx1 - 1][0] == brr[idx2][0] && brr[idx2 - 1][0] == arr[idx1][0]) {
                        ans++;
                    }
                }
            }

            if (r1 < r2) {
                cnt1 = r1 + 1;
                idx1++;
            } else if (r1 > r2) {
                cnt2 = r2 + 1;
                idx2++;
            } else {
                cnt1 = r1 + 1;
                cnt2 = r2 + 1;
                idx1++;
                idx2++;
            }
        }

        System.out.println(ans);

    }

}

```

---

## [E. 小红的树构造](https://ac.nowcoder.com/acm/contest/62033/E)

要让因子少，需要构造的路径越短越好。

那这是什么样的树呢？ 其实sample已经给出了明示，就是 **菊花图**

既然树的形状确定了，那2,3的分布问题呢？

其实这和2,3没啥关系了，就是那个节点数多，就占据中间那个位置。

然后分类讨论，不同的路径。

长度为3的路径,长度为2的路径。

```java
import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;
import java.util.StringTokenizer;

public class Main {

    public static final long mod = 10_0000_0007l;

    public static long qpow(long b, long v) {
        long r = 1l;
        while (v > 0) {
            if (v % 2== 1) {
                r = r * b % mod;
            }
            v /= 2;
            b = b * b % mod;
        }
        return r;
    }

    public static void main(String[] args) {
        
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        // 脑筋急转弯, 菊花图
        long n = sc.nextInt(), k = sc.nextInt();

        if (k == 0 || k == n) {
            long r1 = qpow(4, (n - 1) * (n - 2) / 2) %mod;
            long r2 = qpow(3, n - 1) % mod;
            long r = r1 * r2 % mod;
            System.out.println(r);
        } else {
            long rk = n - k;

            if (k > rk) {
                long t = k;
                k = rk;
                rk = t;
            }

            // 保证k个小, rk大
            long r1 = qpow(4, (rk - 1) * (rk - 2) / 2) %mod;
            long r2 = qpow(6, k * (k - 1) / 2) % mod;
            long r3 = qpow(6, k * (rk - 1)) % mod;

            long r4 = qpow(3, rk - 1) % mod;
            long r5 = qpow(4, k) % mod;

            long r = r1 * r2 % mod * r3 % mod * r4 % mod * r5 %mod;
            System.out.println(r);
        }

    }

}

```

---

# 写在最后

若知是梦何须醒，不比真如一相会。

![alt](https://uploadfiles.nowcoder.com/images/20230730/446702330_1690722214944/D2B5CA33BD970F64A6301FA75AE2EB22)



