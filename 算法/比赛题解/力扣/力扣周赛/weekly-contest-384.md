
# 第 384 场周赛 解题报告 | 珂学家 | 贪心构造 + KMP板子

---
# 前言

---
# 整体评价

因为是新春过年，所以题目出的相对简单一些，T4和上周一样，是字符串匹配模板题。

---
## [T1. 修改矩阵](https://leetcode.cn/contest/weekly-contest-384/problems/modify-the-matrix/)

思路: 模拟

按要求模拟即可

```java []
class Solution {
    public int[][] modifiedMatrix(int[][] matrix) {
        
        int h = matrix.length;
        int w = matrix[0].length;
        int[] cols = new int[w];
        Arrays.fill(cols, Integer.MIN_VALUE);
        for (int j = 0; j < w; j++) {
            for (int i = 0; i < h; i++) {
                cols[j] = Math.max(cols[j], matrix[i][j]);
            }
        }

        int[][] res = new int[h][w];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                res[i][j] = matrix[i][j];
                if (matrix[i][j] == -1) {
                    res[i][j] = cols[j];
                }
            }
        }
        return res;
    }
}
```

---
## [T2. 匹配模式数组的子数组数目 I](https://leetcode.cn/contest/weekly-contest-384/problems/number-of-subarrays-that-match-a-pattern-i/)

和T4一起讲

---
## [T3. 回文字符串的最大数量](https://leetcode.cn/contest/weekly-contest-384/problems/maximum-palindromes-after-operations/)

思路: 贪心构造

回文只求对称的字符相等，其实和字符是啥没关系。

所以这题可以先打散，把字符频率先统计出来。

然后把偶数项合并，奇数项单独留1。

然后按词长度从小到大进行构造(贪心)。

因为这样的思路，能保证最终的结果不会变得更差。

因此这个贪心思路是可以保证的。

```java[]
class Solution {
    public int maxPalindromesAfterOperations(String[] words) {
        
        int[] hash = new int[26];
        for (String w: words) {
            for (char c: w.toCharArray()) {
                hash[c - 'a']++;
            }
        }
        int n1 = 0, n2 = 0;
        for (int i = 0; i < 26; i++) {
            int t = hash[i] % 2;
            n1 += t;
            n2 += (hash[i] - t);
        }
        
        List<Integer> lists = new ArrayList<>();
        for (String w: words) {
            lists.add(w.length());
        }
        Collections.sort(lists);
        
        int res = 0;
        for (int v: lists) {
            if (v % 2 == 0) {
                if (n2 >= v) {
                    n2 -= v;
                    res++;
                }
            } else {
                if (n2 >= v - 1 && n1 > 0) {
                    n2 -= (v - 1);
                    n1 -= 1;
                    res++;
                } else if (v == 1 && n1 > 0) {
                    n1 -= 1;
                    res++;
                } else if (n2 >= v) {
                    n2 -= v;
                    res++;
                } 
            }
        }
        return res;
    }
}
```

--- 

## [T4. 匹配模式数组的子数组数目 II](https://leetcode.cn/contest/weekly-contest-384/problems/number-of-subarrays-that-match-a-pattern-ii/)

思路: 字符串匹配

算法: KMP/Z函数/字符串哈希

因为匹配的是大小关系，与具体的差值无关。可以把大小关系转化为固定的字符串。

那这题就变成很纯粹的字符串匹配题，用板子即可。

这边采用KMP解法

```java[] 
class Solution {
 
    public int countMatchingSubarrays(int[] nums, int[] pattern) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < nums.length - 1; i++) {
            int d = nums[i + 1] - nums[i];
            if (d > 0) sb.append('A');
            else if (d < 0) sb.append("C");
            else sb.append('B');
        }
        StringBuilder sb2 = new StringBuilder();
        for (int v: pattern) {
            if (v > 0) sb2.append('A');
            else if (v == 0) sb2.append('B');
            else sb2.append('C');
        }
        
        List<Integer> lists = KMP.findOccurrences(sb.toString(), sb2.toString());
        return lists.size();
    }
    
    static
    class KMP {

        // *) kmp
        // 在text中，找到pattern出现的位置列表
        public static List<Integer> findOccurrences(String text, String pattern) {
            String cur = pattern + '#' + text;
            int sz1 = text.length(), sz2 = pattern.length();
            List<Integer> v = new ArrayList<>();
            int[] lps = prefixFunction(cur);
            for (int i = sz2 + 1; i <= sz1 + sz2; i++) {
                if (lps[i] == sz2) v.add(i - 2 * sz2);
            }
            return v;
        }

        private static int[] prefixFunction(String ss) {
            char[] s = ss.toCharArray();
            int n = s.length;
            int[] pi = new int[n];
            for (int i = 1; i < n; i++) {
                int j = pi[i - 1];
                while (j > 0 && s[i] != s[j]) j = pi[j - 1];
                if (s[i] == s[j]) j++;
                pi[i] = j;
            }
            return pi;
        }

    }
    
}
```

---

# 写在最后

