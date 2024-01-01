
# 第 378 场周赛 解题报告 | 珂学家 | 分类讨论场

---
# 前言

---
# 整体评价

感觉是分类讨论场，t3用二分，是因为二分不会错，直接分类讨论容易WA.

t4一开始看错题了，T_T, 看成翻转，写了半天StringHash, 还用上双hash，共8个StringHash。

重排的话，其实统计即可，使用26个前缀和，不过需要分类讨论，交集的情况相对麻烦。

---

## [T1. 检查按位或是否存在尾随零](https://leetcode.cn/contest/weekly-contest-378/problems/check-if-bitwise-or-has-trailing-zeros/)

思路: 找规律

核心为

$$ 偶数个数至少为2 $$

```java []
class Solution {
    public boolean hasTrailingZeros(int[] nums) {
        int cnt = 0;
        for (int v: nums) {
            if (v% 2 == 0) cnt++;
        }
        return cnt >= 2;
    }
}
```

---

## [T2. 找出出现至少三次的最长特殊子字符串 I](https://leetcode.cn/contest/weekly-contest-378/problems/find-longest-special-substring-that-occurs-thrice-i/)

和T3一起讲

---

## [T3. 找出出现至少三次的最长特殊子字符串 II](https://leetcode.cn/contest/weekly-contest-378/problems/find-longest-special-substring-that-occurs-thrice-ii/)


思路: 同字母分组，然后二分check/top-3分类讨论

利用滑窗，先按同字母进行分组，这样就可以得到连续字母的数字

其实上，拿到某个字母的最大段为m，其答案至少为m-2

这样枚举的话，时间复杂度为$O(n)$

```java []
class Solution {

    public int maximumLength(String s) {

        char[] str = s.toCharArray();
        Map<Integer, List<Integer>> hash = new HashMap<>();

        int i = 0;
        while (i < str.length) {
            int p = str[i] - 'a';
            int j = i + 1;
            while (j < str.length && str[j] == str[i]) {
                j++;
            }

            int cnt = j - i;
            hash.computeIfAbsent(p, x -> new ArrayList<>()).add(cnt);
            i = j;
        }

        int res = 0;
        for (var kv: hash.entrySet()) {
            List<Integer> lists = kv.getValue();
            Collections.sort(lists, Comparator.comparing(x -> -x));


            int l = 0, r = lists.get(0);
            while (l <= r) {
                int m = l + (r - l) / 2;

                int cnt = 0;
                for (int v: lists) {
                    if (v == m) cnt++;
                    else if (v == m + 1) cnt += 2;
                    else if (v >= m + 2) cnt += 3;
                }
                if (cnt >= 3) {
                    l = m + 1;
                } else {
                    r = m - 1;
                }
            }

            res = Math.max(res, r);
        }

        return res == 0 ? -1 : res;
    }

}
```

---

## [T4. 回文串重新排列查询](https://leetcode.cn/contest/weekly-contest-378/problems/palindrome-rearrangement-queries/)

思路: 对于回文，其实可以逆转后半段，区间重排是否相等，可以借助前缀和比较字母数即可

因为这题查询量大，而且很难离线处理。

因为这题的思路还是：

$$在线解法$$

那这题的核心就是，如何降低查询的时间复杂度

重排后相等，只要计数相等就行，因此按字母(26)前缀和求解即可

这题还有一个难点，就是改动区间有交集

```java []
class Solution {

    int[] evalute(int[][] pre, int a, int b) {
        int[] hash = new int[26];
        for (int i = 0; i < 26; i++) {
            hash[i] = pre[i][b + 1] - pre[i][a];
        }
        return hash;
    }

    boolean sub(int[][] pre, int a, int b, int[] hash, int[] hash2) {
        for (int i = 0; i < 26; i++) {
            int t = pre[i][b + 1] - pre[i][a];
            hash[i] -= t;
            hash2[i] -= t;
            if (hash[i] < 0) return false;
        }
        return true;
    }


    public boolean[] canMakePalindromeQueries(String s, int[][] queries) {

        int n = s.length();
        int n2 = n / 2;

        String s1 = s.substring(0, n2);
        char[] str1 = s1.toCharArray();
        String s2 = new StringBuilder(s.substring(n2)).reverse().toString();
        char[] str2 = s2.toCharArray();


        int[][] pre1 = new int[26][n2 + 1];
        int[][] pre2 = new int[26][n2 + 1];

        for (int i = 0; i < 26; i++) {
            for (int j = 0; j < n2; j++) {
                pre1[i][j + 1] = pre1[i][j] + (str1[j] - 'a' == i ? 1 : 0);
                pre2[i][j + 1] = pre2[i][j] + (str2[j] - 'a' == i ? 1 : 0);
            }
        }

        int[] diff = new int[n2 + 1];
        for (int i = 0; i < n2; i++) {
            diff[i + 1] = diff[i] + (str1[i] == str2[i] ? 1 : 0);
        }


        int m = queries.length;
        boolean[] res = new boolean[m];

        for (int i = 0; i < m; i++) {
            int[] q = queries[i];
            int a = q[0], b = q[1], c = q[2], d = q[3];
            int c2 = n - 1 - d;
            int d2 = n - 1 - c;

            boolean ok = true;
            // 分类讨论
            int lMax = Math.max(a, c2);
            int rMin = Math.min(b, d2);
            if (rMin >= lMax) {
                // 区间有交集
                int p1 = Math.min(a, c2);
                int p2 = lMax - 1;
                int p3 = lMax;
                int p4 = rMin;
                int p5 = rMin + 1;
                int p6 = Math.max(b, d2);

                if (p1 - 1 >= 0) {
                    if (diff[p1] != p1) {
                        ok = false;
                    }
                }
                if (p6 + 1 < n2) {
                    if (diff[n2] - diff[p6 + 1] != n2 - p6 - 1) {
                        ok = false;
                    }
                }

                int[] hash1 = evalute(pre1, p1, p6);
                int[] hash2 = evalute(pre2, p1, p6);

                // *)
                if (p2 - p1 >= 0) {
                    if (a == p1) {
                        ok = ok & sub(pre2, p1, p2, hash1, hash2);
                    } else {
                        ok = ok & sub(pre1, p1, p2, hash2, hash1);
                    }
                }
                if (p6 - p5 >= 0) {
                    if (b == p6) {
                        ok = ok & sub(pre2, p5, p6, hash1, hash2);
                    } else {
                        ok = ok & sub(pre1, p5, p6, hash2, hash1);
                    }
                }

                if (!Arrays.equals(hash1, hash2)) {
                    ok = false;
                }
            } else {
                // 区间没交集
                int p1 = Math.min(a, c2);
                int p2 = Math.min(b, d2);
                int p3 = p2 + 1;
                int p4 = Math.max(a, c2) - 1;
                int p5 = Math.max(a, c2);
                int p6 = Math.max(b, d2);

                if (p1 - 1 >= 0) {
                    if (diff[p1] != p1) {
                        ok = false;
                    }
                }
                if (p6 + 1 < n2) {
                    if (diff[n2] - diff[p6 + 1] != n2 - p6 - 1) {
                        ok = false;
                    }
                }
                if (p4 >= p3) {
                    if (diff[p4 + 1] - diff[p3] != p4 - p3 + 1) {
                        ok = false;
                    }
                }

                if (p2 >= p1) {
                    int[] hash1 = evalute(pre1, p1, p2);
                    int[] hash2 = evalute(pre2, p1, p2);
                    if (!Arrays.equals(hash1, hash2)) {
                        ok = false;
                    }
                }
                if (p6 >= p5) {
                    int[] hash1 = evalute(pre1, p5, p6);
                    int[] hash2 = evalute(pre2, p5, p6);
                    if (!Arrays.equals(hash1, hash2)) {
                        ok = false;
                    }
                }
            }

            res[i] = ok;
        }
        return res;
    }

}
```

---

# 写在最后






