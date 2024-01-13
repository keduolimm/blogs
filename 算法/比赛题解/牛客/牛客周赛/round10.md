# 牛客周赛 Round 10 解题报告 | 珂学家 | 三分模板 + 计数DFS + 回文中心扩展

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230904/446702330_1693760773476/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

T2真是一个折磨人的小妖精，写了两版DFS，第二版计数DFS才过。T3是三分模板，感觉也可以求导数。T4的数据规模才n=1000，因此中心扩展的$O(n^2)$当仁不让。

---

## [A. 游游的最长稳定子数组](https://ac.nowcoder.com/acm/contest/64272/A)

滑窗经典题

从某个左端点出发，按顺序找到最远的右端点

然后把该右端点变成新的左端点，继续寻找直至结束

```java []
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) arr[i] = sc.nextInt();

        int ans = 0;
        int j = 0;
        while (j < n) {
            int k = j + 1;
            while (k < n && Math.abs(arr[k] - arr[k - 1]) <= 1) {
                k++;
            }
            ans = Math.max(ans, k - j);
            j = k;
        }

        System.out.println(ans);
    }

}
```

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    int n;
    cin >> n;
    vector<int> arr(n);
    for (int i = 0; i< n; i++) {
        cin >> arr[i];
    }
    
    int res = 1;
    int j = 0;
    while (j < n) {
        int k = j + 1;
        while (k < n && abs(arr[k - 1] - arr[k]) <= 1) {
            k++;
        }
        res = max(res, k - j);
        j = k;
    }
    cout << res << endl;
    
    return 0;
}
```

---

## [B. 游游的字符重排](https://ac.nowcoder.com/acm/contest/64272/B)

暴力搜索好像TLE了，而且需要额外处理去重

那如何搜索，可以不考虑去重的情况呢？

可以对每个字母进行频数统计

然后进行dfs，这样有一个好处，就是天然去重，时间复杂度为$O(n!)$

```java []
import java.io.*;
import java.util.*;

public class Main {

    // 计数DFS
    public static int dfs(int[] nums, int n, int now, int prev) {
        if (now == n) {
            return 1;
        }
        int res = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0 && i != prev) {
                nums[i]--;
                res += dfs(nums, n, now + 1, i);
                nums[i]++;
            }
        }
        return res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        Map<Character, Integer> hash = new HashMap<>();
        for (char c: str) hash.merge(c, 1, Integer::sum);
        int[] nums = hash.values().stream().mapToInt(x -> x).toArray();
        System.out.println(dfs(nums, str.length, 0, -1));
    }

}

```

```c++ []


#include <bits/stdc++.h>

using namespace std;

using int64 = long long;

int64 dfs(const string &s, char last, int mask) {
    int n = s.length();
    if (mask == (1 << n) - 1) {
        return 1LL;
    }
    int64 res = 0;
    for (int i = 0; i < n; i++) {
        if ((mask & (1 << i)) == 0) {
            if (s[i] == last || (i > 0 && s[i] == s[i - 1] && (mask &(1 << (i - 1))) == 0) ) {
                continue;
            }
            res += dfs(s, s[i], mask | (1 << i));
        }
    }
    return res;
}

int main() {
    
    string s;
    cin >> s;
    
    sort(s.begin(), s.end());
    
    int64 res = dfs(s, ' ', 0);
    cout << res << endl;
    
    return 0;
}
```

---

## [C. 游游开车出游](https://ac.nowcoder.com/acm/contest/64272/C)

三分板子题，该函数为凹函数。

> z = y / (v + x*t) + t

该函数先单调递减，然后单调递增

### 1. 三分板子

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static double calc(double y, double x, double v, double t) {
        return y / (v + x * t) + t;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        double v = sc.nextDouble(), x = sc.nextDouble(), y = sc.nextDouble();

        // 三分板子
        double l = 0.0, r = 1e9;
        while (r - l >= 1e-7) {
            // 要求1e-6
            double s = (r - l) / 3.0;
            double t1 = l + s;
            double t2 = l + 2 * s;

            double res1 = calc(y, x, v, t1);
            double res2 = calc(y, x, v, t2);

            if (res1 >= res2) {
                l = t1;
            } else {
                r = t2;
            }
        }

        System.out.println(calc(y, x, v, l));

    }

}
```
---

### 2. 求导

f(t) = y / (v + x*t) + t

求导

f‘(t) = -xy / (v + x*t)^2 + 1

求f'(t) = 0 的解

x^2 * t^2 + 2*v*x*t + v^2 - xy = 0;     t为变量， x,y,v都是常数

t'=(-v +/- sqrt(xy)) / x,

> 不知道，牛客啥时候对latex语法不那么友好了，写起来难受，看的人更难受

因为xy>0， 所以一定有解，同时 t>=0, 因此只有一个可能解

t' = (-v + sqrt(xy)) / x

t' = max(0, t'), 保证t>=0

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static double calc(double y, double x, double v, double t) {
        return y / (v + x * t) + t;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        double v = sc.nextDouble(), x = sc.nextDouble(), y = sc.nextDouble();

        // 求一元二次方程的根
        double t = (-v + Math.sqrt(x*y)) / x;
        // 保证t>=0
        t = Math.max(0, t);
        System.out.println(calc(y, x, v, t));
    }

}
```








---

## [D. 游游的回文子串](https://ac.nowcoder.com/acm/contest/64272/D)

因为n=1000，所以中心扩展就好了

- 同一个区域, 长度为n, 方案数n*(n+1)/2
- 以某个区域为中心，往两边扩展，则线性增长

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr = new long[n];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = sc.nextInt();
        }

        long ans = 0;
        long mod = 10_0000_0007l;
        for (int i = 0; i < n; i++) {
            // 枚举某个区域
            long t = arr[i] * (arr[i] + 1) / 2;
            ans = (ans + t % mod) % mod;

            // 中心扩展
            for (int j = 1; i - j >= 0 && i + j < n; j++) {
                if (arr[i - j] != arr[i + j]) {
                    // 长度不等，取最小，通过跳出
                    ans = (ans + Math.min(arr[i - j], arr[i + j])) % mod;
                    break;
                } else {
                    ans = (ans + arr[i - j]) % mod;
                }
            }
        }
        System.out.println(ans);

    }

}

```

---

# 写在最后


![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230904/446702330_1693760785885/D2B5CA33BD970F64A6301FA75AE2EB22)
