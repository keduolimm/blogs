
# 牛客小白月赛81 解题报告 | 珂学家 | 期望 + 二分验证 + 变动点压缩

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231121/446702330_1700573551344/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

D题感觉像博弈论书的经常提到的一个场景题。

E题题意有点晦涩，赛后才看明白啥意思，F题是道经典题吧，但是感觉太追求最优解了。

ST预处理，查询为分治二分，时间复杂度为$O(nlogn)$ + $O(m * 32logn)$。

正解预处理，维护每个右端点的左侧变动列表，而下一个右端点基于相邻点构建，所以为$O(32n)$ + $O(32m)$。

---

## [A. 小辰打比赛](https://ac.nowcoder.com/acm/contest/69791/A)

田忌赛马

优先选择比自己实力弱进行pk，这样成就感会到达某个高点。

这题其实可以不用排序

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), s = sc.nextInt();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        Arrays.sort(arr);
        
        int res = 0;
        for (int i = 0; i < n; i++) {
            if (s <= arr[i]) break;
            res += arr[i];
        }
        System.out.println(res);
    }

}
```

---

## [B. 小辰的圣剑](https://ac.nowcoder.com/acm/contest/69791/B)

声望增加bi，和 是否杀怪物有点， 和当天杀怪物的多少无关。

如果声望和怪物数有关，那感觉这题巨难。

这样的话，就容易写了，可以$O(n^2)$实现

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long m = sc.nextLong(), u = sc.nextLong();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        int[] brr = new int[n];
        for (int i = 0; i < n; i++) {
            brr[i] = sc.nextInt();
        }

        int ans = 0;
        for (int i = 0; i < n; i++) {
            long s1 = 0, s2 = 0;
            for (int j = i; j < n; j++) {
                s1 += arr[j];
                s2 += brr[j];
                if (s1 > m || s2 > u) break;
                ans = Math.max(ans, j - i + 1);
            }
        }
        System.out.println(ans);
    }

}
```


---

## [C. 陶陶学算术](https://ac.nowcoder.com/acm/contest/69791/C)

这是道好题

可以对每一个乘数/除数，进行质因子拆解，然后放入到一个map中，乘数为加，除数为减

如果两个操作序列最终结果相等，那么其hash所有的key/value应该全相等。


```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static List<int[]> split(int v) {
        List<int[]> res = new ArrayList<>();
        for (int i = 2; i <= v / i; i++) {
            if (v % i == 0) {
                int cnt = 0;
                while (v % i == 0) {
                    v /= i;
                    cnt++;
                }
                res.add(new int[] {i, cnt});
            }
        }
        if (v > 1) res.add(new int[] {v, 1});
        return res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        Map<Integer, Integer> hash1 = new HashMap<>();
        int sign1 = 1;
        for (int i = 0; i < n; i++) {
            int op = sc.nextInt();
            int x = sc.nextInt();
            if (x < 0) {
                x = -x;
                sign1 = -sign1;
            }
            List<int[]> factors = split(x);
            if (op == 1) {
                for (int[] e: factors) {
                    hash1.merge(e[0], e[1], Integer::sum);
                }
            } else {
                for (int[] e: factors) {
                    hash1.merge(e[0], -e[1], Integer::sum);
                }
            }
        }

        int m = sc.nextInt();
        Map<Integer, Integer> hash2 = new HashMap<>();
        int sign2 = 1;
        for (int i = 0; i < m; i++) {
            int op = sc.nextInt();
            int x = sc.nextInt();
            if (x < 0) {
                x = -x;
                sign2 = -sign2;
            }
            List<int[]> factors = split(x);
            if (op == 1) {
                for (int[] e: factors) {
                    hash2.merge(e[0], e[1], Integer::sum);
                }
            } else {
                for (int[] e: factors) {
                    hash2.merge(e[0], -e[1], Integer::sum);
                }
            }
        }

        if (sign1 != sign2) {
            System.out.println("NO");
        } else {
            boolean ok = true;
            for (Map.Entry<Integer, Integer> kv: hash1.entrySet()) {
                int v2 = hash2.getOrDefault(kv.getKey(), 0);
                if (kv.getValue() != v2) {
                    ok = false;
                    break;
                }
            }
            for (Map.Entry<Integer, Integer> kv: hash2.entrySet()) {
                int v2 = hash1.getOrDefault(kv.getKey(), 0);
                if (kv.getValue() != v2) {
                    ok = false;
                    break;
                }
            }
            System.out.println(ok ? "YES" : "NO");
        }
    }

}
```

---

## [D. 小辰的借钱计划](https://ac.nowcoder.com/acm/contest/69791/D)

这是一道期望题

其实是枚举a的因子和倍数，求总期望E

- E > a， 则交换
- E <= a, 则不交换

很像一道博弈论出来的经典场景题。

a的因子个数，可以在$\sqrt{a}$求解，倍数的话，可以$O(1)$快速搞定

据说原本该题是B题的位子，后面因为期望的历史战绩原因被挪到了D的位置。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    // 求因子和，因子总数
    public static int[] factors(int v) {
        int res = 0;
        int cnt = 0;
        for (int i = 1; i <= v / i; i++) {
            if (v % i == 0) {
                if (v / i == i) {
                    res += i;
                    cnt++;
                } else {
                    res += i;
                    res += v / i;
                    cnt += 2;
                }
            }
        }
        return new int[] {res, cnt};
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int t = sc.nextInt();
        while (t-- > 0) {
            int m = sc.nextInt(), a = sc.nextInt();

            if (2 * a >= m) {
                System.out.println("NO");
            } else {
                // 倍数的个数(包含a)
                int n1 = ((m - a) / a);
                int e1 = (n1 + 1) * n1 / 2 * a;

                // 因子的个数(包含a)
                int[] tmp = factors(a);
                int n2 = tmp[1];
                int e2 = tmp[0];

                // 不会int溢出
                if (e1 + e2 > a * (n1 + n2)) {
                    System.out.println("YES");
                } else {
                    System.out.println("NO");
                }
            }
        }
    }

}
```


---

## [E. 小辰的智慧树](https://ac.nowcoder.com/acm/contest/69791/E)

题意多少有点晦涩了，case仅有一个，且无解释，泪目。

这题的关键是理解 $x * (h' * 2 - x)$ ， $h' = h' - x$

我们假设, 执行x次操作，每次为1

则收益为 $\sum_{i=0}^{i=x-1} (h' - i) \times 2 - 1 = 2 \times x \times h' - x^2$

也就是说对于某棵智慧树，

> 其收益只和最终被截去的x有关，和过程无关。

这一点很重要，假设每次取1大小，那么h最大，收益越大。

因此这题到现在很容易了，假设每次取1的高度，那每次贪心削最高的智慧树。

---

因为数据范围很大，没办法这样贪心模拟。

所以关键是什么呢？

可以二分枚举 被全部削掉的高度（限制的智慧树除外）。

找到这个高度点h后，如果还有多余的，那一定是h-1高的，削掉1高度。

类似的题，好像做过好几次了，排序后模拟找到分界点也可以，不过不如二分直接。

```java
import java.io.*;
import java.util.*;
import java.util.function.Function;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt();
        long m = sc.nextLong();

        int mz = 0;
        int[][] ps = new int[n][2];
        for (int i = 0; i < n; i++) {
            ps[i][0] = sc.nextInt();
            ps[i][1] = sc.nextInt();
            mz = Math.max(ps[i][0] + 1, mz);
        }

        // 定义评估函数
        Function<Integer, long[]> evaluator = (h) -> {
            long y = 0;
            long acc = 0;
            for (int i = 0; i < n; i++) {
                if (ps[i][0] >= h) {
                    long x = ps[i][0] - Math.max(h, ps[i][1]);
                    y += x;
                    acc += x * (ps[i][0] * 2 - x);
                }
            }
            return new long[] {acc, y};
        };

        // 二分
        int l = 0, r = mz;
        while (l <= r) {
            int mid = l + (r - l) / 2;

            long[] cur = evaluator.apply(mid);
            long y = cur[1];
            if (y > m) {
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }

        long[] cur = evaluator.apply(l);
        long res = cur[0];
        long y = cur[1];
        // 补上剩余的
        if (l > 0 && m > y) {
            res += (m - y) * (l * 2 - 1);
        }

        System.out.println(res);
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

## [F. 小辰刚学 gcd](https://ac.nowcoder.com/acm/contest/69791/E)

> 我多么想，出题人，你可以亚撒西一点。

关于大量区间查询问题，还有gcd,最值，位运算这类，往往可以借助ST来加速。

这题比较特殊，可以分治二分（不知道这样说法对不对）来快速计算整个区间的不同gcd数。

可惜这个思路TLE了，感觉被严格卡复杂度了，多一个log也不行。

Java卡在56%， 等价C++（常数优化后）卡90%。

---

其实正解的思路，在数组 按与/或操作，区间查询最值等等问题，经常有出现。

> 就是维护变动的点列表

位运算的话，最多32位，所以最多32个点

而gcd的话，其实也是一样，因为最小的质因子为2， 而2^32是最大的整数

而arr[i + 1]变动点的列表，可以基于arr[j]的基础上，生成。

因此整个是时间复杂度为 $O(32n)$

---

贴一个chatgpt翻译后的c++代码

原一模一样的java代码被卡在70%, 数据还是卡常了。

```c++
#include <iostream>  
#include <vector>  
#include <algorithm>  
#include <cmath>  
  
using namespace std;  
  
long gcd(long a, long b) {  
    return b == 0 ? a : gcd(b, a % b);  
}  
  
int main() {  
    
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    
    int n, q;  
    cin >> n >> q;  
    vector<long> arr(n);  
    vector<vector<pair<long, int>>> g(n);  
    for (int i = 0; i < n; i++) {  
        cin >> arr[i];  
        long v = arr[i];  
        g[i].push_back({v, i});  
        if (i > 0) {  
            auto& prev = g[i - 1];  
            for (auto& pair : prev) {  
                long tv = gcd(v, pair.first);  
                if (tv != v) {  
                    g[i].push_back({tv, pair.second});  
                }  
                v = tv;  
                if (v == 1) break;  
            }  
        }  
    }  
    vector<int> res;  
    for (int i = 0; i < q; i++) {  
        int l, r;  
        cin >> l >> r;  
        l--;  // adjust for zero-based indexing  
        r--;  // adjust for zero-based indexing  
        auto& rightList = g[r];  
        int left = 0, right = rightList.size() - 1;  
        while (left <= right) {  
            int mid = left + (right - left) / 2;  
            if (rightList[mid].second >= l) {  
                left = mid + 1;  
            } else {  
                right = mid - 1;  
            }  
        }  
        res.push_back(left);  
    }  
    for (auto& val : res) cout << val << "\n";  
    return 0;  
}
```

----

# 写在最后
![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231121/446702330_1700573585074/D2B5CA33BD970F64A6301FA75AE2EB22)



