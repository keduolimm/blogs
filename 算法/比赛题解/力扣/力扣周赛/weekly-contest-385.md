

---
# 第 385 场周赛 解题报告 | 珂学家 | 字典树专场

---
# 前言

---

# 整体评价

这是一场字典树专场，除了t3这个小模拟之外，1，2，4皆可用字典树搞定。

T4感觉做法挺多的，其实，但是字典树应该效率最高的。

---

## [T1. 统计前后缀下标对 I](https://leetcode.cn/contest/weekly-contest-385/problems/count-prefix-and-suffix-pairs-i/)

思路: 模拟

$O(n^2)$全遍历即可

```java []
class Solution {
    public int countPrefixSuffixPairs(String[] words) {
        int res = 0;
        for (int i = 0; i < words.length; i++) {
            for (int j = 0; j < i; j++) {
                if (words[i].startsWith(words[j]) && words[i].endsWith(words[j])) {
                    res ++;
                }
            }
        }
        return res;
    }
}
```

---

## [T2. 最长公共前缀的长度](https://leetcode.cn/contest/weekly-contest-385/problems/find-the-length-of-the-longest-common-prefix/)

思路: 字典树

这样的时间复杂度为 

$$ (max_{ai} len(word_{ai})) * (\sum_{bi} len(word_{bi}))$$

因为第一组的数字最大为8位数，因此其深度最多为8

```java []
public class Solution {
    
    static class Trie {
        Trie[] cs = new Trie[26];
        void add(String w) {
            Trie cur = this;
            for (char c: w.toCharArray()) {
                int p = c - '0';
                if (cur.cs[p] == null) {
                    cur.cs[p] = new Trie();
                }
                cur = cur.cs[p];
            }
        }
        int query(String w) {
            Trie cur = this;
            int res = 0;
            for (char c: w.toCharArray()) {
                int p = c - '0';
                if (cur.cs[p] == null) {
                    return res;
                }
                cur = cur.cs[p];
                res++;
            }
            return res;
        }
    }
    
    public int longestCommonPrefix(int[] arr1, int[] arr2) {
        Trie root = new Trie();
        for (int v: arr1) {
            root.add(String.valueOf(v));
        }
        int res = 0;
        for (int v: arr2) {
            int tmp = root.query(String.valueOf(v));
            res = Math.max(tmp, res);
        }
        return res;
    }
    
}
```

---

## [T3. 出现频率最高的素数](https://leetcode.cn/contest/weekly-contest-385/problems/most-frequent-prime/)

思路: 模拟

知识点: 质数筛，质数判定

按题目模拟执行即可

```java []
class Solution {
    
    static int[][] dirs = new int[][] {
        {-1, 0}, {1, 0}, {0, -1}, {0, 1},
        {-1, -1}, {-1, 1}, {1, -1}, {1, 1}
    };
    
    List<Integer> create(int[][] mat, int y, int x) {
        
        List<Integer> res = new ArrayList<>();
        int n = mat.length, m = mat[0].length;
        for (int i = 0; i < dirs.length; i++) {
            int v = mat[y][x];
            int j = 0;
            
            int ty = y, tx = x;
            int base = 10;
            do {
                ty += dirs[i][0];
                tx += dirs[i][1];
                if (ty >= 0 && ty < n && tx >= 0 && tx < m) {
                    v = v + mat[ty][tx] * base;
                } else {
                    break;
                }
                base *= 10;
                res.add(v);
            } while(true);
        }
        
        return res;
    }
    
    boolean isPrime(int v) {
        if (v <= 1) return false;
        for (int i = 2; i <= v / i; i++) {
            if (v % i == 0) return false;
        }
        return true;
    }
    
    public int mostFrequentPrime(int[][] mat) {
        Map<Integer, Integer> hash = new HashMap<>();
        int n = mat.length, m = mat[0].length;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                List<Integer> tmp = create(mat, i, j);
                for (int v: tmp) {
                    hash.merge(v, 1, Integer::sum);
                }
            }
        }    
        
        int fz = -1;
        int ans = -1;
        
        for (var kv: hash.entrySet()) {
            if (isPrime(kv.getKey()) && kv.getKey() > 10) {
                if (fz < kv.getValue() || (fz == kv.getValue() && ans < kv.getKey())) {
                    fz = kv.getValue();
                    ans = kv.getKey();
                }
            }
        }
           
        return ans;
        
    }
    
}
```

---

## [T4. 统计前后缀下标对 II](https://leetcode.cn/contest/weekly-contest-385/problems/count-prefix-and-suffix-pairs-ii/)

思路: 字典树+z函数

因为涉及到前后缀一致的问题

所以必然需要引入 

- z函数
- stringhash

这类技巧

有涉及到前缀问题，自然就引出字典树

```java []
class Solution {

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

    static class Trie {
        Trie[] cs = new Trie[26];
        int rel;
        
        void add(String w) {
            Trie cur = this;
            for (char c: w.toCharArray()) {
                int p = c - 'a';
                if (cur.cs[p] == null) {
                    cur.cs[p] = new Trie();
                }
                cur = cur.cs[p];
            }
            cur.rel++;
        }

        int count(String w) {
            Trie cur = this;
            int[] zz = zFunction(w);

            int res = 0;
            int n = w.length();
            for (int i = 0; i < n; i++) {
                int p = w.charAt(i) - 'a';
                if (cur.cs[p] == null) {
                    return res;
                }
                cur = cur.cs[p];
                if (zz[n - 1 - i] == i + 1) {
                    res += cur.rel;
                }
            }
            return res;
        }
    }

    public long countPrefixSuffixPairs(String[] words) {
        long res = 0;
        Trie root = new Trie();
        for (String w: words) {
            res += root.count(w);
            root.add(w);
        }
        return res;
    }

}
```

---

### 特别的Trie[补充]

```java []

```

---

# 写在最后



