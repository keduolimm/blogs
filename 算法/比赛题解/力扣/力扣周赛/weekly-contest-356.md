# 第 356 场周赛 解题报告 | 珂学家 | Z函数优化 + 另一种形态的数位DP



---

# 前言

![image.png](https://pic.leetcode.cn/1690821304-LROcmm-image.png)

剑，和茶一样，只有细细品味，才能理解它的风雅。

---

# 整体评价

T2是道滑窗题，T3是道模拟题，感觉简单但是超级易错(数据范围小), T4是道经典的位数DP题。

---

## [T1. 满足目标工作时长的员工数目](https://leetcode.cn/problems/number-of-employees-who-met-the-target/)

据说诚意满满，毫无陷阱

```java
class Solution {
    public int numberOfEmployeesWhoMetTarget(int[] hours, int target) {
        return (int)Arrays.stream(hours).filter(x -> x >= target).count();
    }
}
```

---

## [T2. 统计完全子数组的数目](https://leetcode.cn/problems/count-complete-subarrays-in-an-array/)

写个滑窗算法，一般就是遍历固定右端点，然后滑动左端点。

由于区间$[left, right]$是不满足条件，因此每次累加 left (从0计数)

```java
class Solution {
    
    public int countCompleteSubarrays(int[] nums) {

        int cnt = (int)Arrays.stream(nums).distinct().count();
        
        int ans = 0;
        int n = nums.length;
        
        int j = 0;
        Map<Integer, Integer> range = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            range.merge(nums[i], 1, Integer::sum);
            while (j <= i && range.size() == cnt) {
                range.computeIfPresent(nums[j++], (k, v) -> v > 1 ? v - 1 : null);
            }
            ans += j;
        }
        
        return ans;
    }
    
}
```


---

## [T3. 包含三个字符串的最短字符串](https://leetcode.cn/problems/shortest-string-that-contains-three-strings/)

对于字符串的题，真的没辙。
幸好这题的字符串长度为100，字符串数量为3.

对于字符串，采用全排列枚举$3!$，而字符串匹配问题，可以采用最朴素的解法。

当然也可以采用 $kmp / string\ hash$ 方法，以$O(n)$的代价来快去匹配。

这里其实还有思维过程

> 枚举排列贪心所得的解中一定包含最优解

2个字符串一定成立，3个字符串时，从最优解的(3个字符串位置)反推

可以思考下
> 如果这题有4个字符串$a, b, c, d$, 长度限制为100，那如何解呢？ 上述结论还成立不？


```java
class Solution {
    
    boolean check(char[] buf, int i, char[] p) {
        for (int s = i; s - i < p.length && s < buf.length; s++) {
            if (buf[s] != p[s - i]) return false;
        }
        return true;
    }

    public String merge(String a, String b) {
        char[] astr = a.toCharArray();
        char[] bstr = b.toCharArray();
        for (int i = 0; i <= astr.length; i++) {
            if (check(astr, i, bstr)) {
                String r1 = new String(astr, 0, i);
                String r2 = new String(bstr);
                String r3 = (bstr.length < (astr.length - i)) ? 
                    new String(astr, i + bstr.length, astr.length - i - bstr.length) : "";
                return r1 + r2 + r3;
            }
        }
        // unreachable 
        return null;
    }

    public String solve(String a, String b, String c) {
        return merge(merge(a, b), c);
    }

    public String minimumString(String a, String b, String c) {
        // 手工枚举所有的情况
        var list = List.of(
                solve(a, b, c), solve(a, c, b),
                solve(b, a, c), solve(b, c, a),
                solve(c, a, b), solve(c, b, a)
        );
        
        return list.stream().min(Comparator.comparingInt(String::length).thenComparing(x -> x)).get();
    }

}
```

### Z函数优化

这是IO-Wiki关于Z函数的解释

[https://oi-wiki.org/string/z-func/](https://oi-wiki.org/string/z-func/)

核心就是构造

> b + "#" + a

$b为模式串，a为被匹配的串$

```java
class Solution {

    // z函数模板, https://oi-wiki.org/string/z-func/
    static int[] zFunction(String a) {
        char[] s = a.toCharArray();
        int n = s.length;
        int[] z = new int[n];
        z[0] = n;
        for (int i = 1, l = 0, r = 0; i < n; ++i) {
            if (i <= r && z[i - l] < r - i + 1) {
                z[i] = z[i - l];
            } else {
                z[i] = Math.max(0, r - i + 1);
                while (i + z[i] < n && s[z[i]] == s[i + z[i]]) ++z[i];
            }
            if (i + z[i] - 1 > r) {
                l = i;
                r = i + z[i] - 1;
            }
        }
        return z;
    }

    public String merge(String a, String b) {
        String s = b + "#" + a;
        int[] z = zFunction(s);
        for (int i = 0; i < s.length(); i++) {
            if (z[i] == b.length()) {
                return a;
            } else if (s.length() - i <= b.length() && z[i] == s.length() - i) {
                return a + b.substring(s.length() - i);
            }
        }
        return a + b;
    }

    public String minimumString(String a, String b, String c) {
        // 生成排列, 通过排列编号 逆向生成 第k小排列
        return IntStream.range(0, 6).mapToObj(x -> {
                var list = new ArrayList<>(List.of(a, b, c));
                return merge(merge(list.remove(x / 2), list.remove(x % 2)), list.get(0));
            }).min(Comparator.comparingInt(String::length).thenComparing(x -> x)).get();
    }
    
}
```

----

## [T4. 统计范围内的步进数字数目](https://leetcode.cn/problems/count-stepping-numbers-in-range/)

写一下另一种形态数位DP解法

位数DP难在边界，因此该方法针对边界问题，分为两部分。

1. 沿着上边界$S$顺序遍历，累加满足要求的解数
2. 长度不足$|S|$，单独求解累加

> 和递归写法的位数DP模板相比，其最大的优势就是跑的快, ^_^。

回到题目，需要做一些预处理

$对于长度n, 首位元素是i的步进数字个数，定义为opt[n][i]$

可以预处理100位长度

算下时间复杂度, $n为整数的长度$

- 预处理opt[n][i]，$10 * n$
- 第一部分，$10 + (n-1) * 2$
- 第二部分, $9 * (n - 1)$

所以整个时间复杂度为 $O(c*n)，c是常数，n为整数的长度$,

```java
class Solution {

    long mod = 10_0000_0007l;

    // 长度为i,首元素为j的总项数
    Long[][] opt;

    void init(int n) {
        opt = new Long[n + 1][10];
        for (int i = 0; i < 10; i++) {
            opt[1][i] = 1l;
        }
        for (int i = 1; i < n; i++) {
            opt[i + 1][0] = opt[i][1];
            for (int j = 1; j < 9; j++) {
                opt[i + 1][j] = (opt[i][j - 1] + opt[i][j+1]) % mod;
            }
            opt[i + 1][9] = opt[i][8];
        }
    }

    long compute(String v) {

        long res = 0;

        int n = v.length();
        boolean found = true;

        // 第一部分，沿着上边界S顺序遍历，累加满足的解数
        int first = v.charAt(0) - '0';
        //      处理0位置，[1, first)直接累加
        for (int i = 1; i < first; i++) {
            res += opt[n][i];
            res %= mod;
        }
        //      沿着[1, n)位处理
        int prev = first;
        for (int i = 1; i < n; i++) {
            int p = v.charAt(i) - '0';
            if (p > prev + 1) res += opt[n - i][prev + 1];
            if (p > prev - 1 && prev >= 1) res += opt[n - i][prev - 1];
            res %= mod;

            if (p == prev + 1 || p == prev - 1) {
                prev = p;
            } else {
                found = false;
                break;
            }
        }
        //       末尾这项千万别忽略
        if (found) res = (res + 1) % mod;


        // 第2部分, 长度不足$|S|$，单独求解累加
        //      统计长度[1, n - 1]的所有满足的项
        for (int i = n - 1; i >= 1; i--) {
            for (int j = 1; j < 10; j++) {
                res += opt[i][j];
                res %= mod;
            }
        }
        return res;
    }

    boolean specail(String s) {
        for (int i = 0; i < s.length() - 1; i++) {
            if (Math.abs(s.charAt(i) - s.charAt(i + 1)) != 1) return false;
        }
        return true;
    }

    public int countSteppingNumbers(String low, String high) {
        // 很常见的写法
        init(high.length());

        long r1 = compute(high);
        long r2 = compute(low);
        // 特判low, 这个比low - 1要友好
        long r3 = specail(low) ? 1 : 0;

        return (int)(((r1 - r2 + r3) % mod + mod) % mod);
    }

}
```

----

# 写在最后

若知是梦何须醒，不比真如一相会。

![image.png](https://pic.leetcode.cn/1690821232-XfzkjM-image.png)