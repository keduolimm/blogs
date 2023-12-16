# 第 355 场周赛 解题报告 | 珂学家 | 异或树技巧


---

# 前言

![image.png](https://pic.leetcode.cn/1690089958-PsWuZa-image.png)

你我同为异世界之旅人，在此相遇皆为命运之意志。

---

# 整体评价

一看难度分布，就怂了，哈哈。

T3 Debug到崩溃， T4倒是经典异或树技巧，以前做过反而觉得简单。

---

## [T1. 按分隔符拆分字符串](https://leetcode.cn/contest/weekly-contest-355/problems/split-strings-by-separator/)

模拟就行，但是Java的字符串函数$split，replace$参数好像是正则匹配相关，对于特殊符号".|"就很麻烦。

因此采用最笨笨的方法，遍历替换。

```java
class Solution {
    public List<String> splitWordsBySeparator(List<String> words, char separator) {
        
        List<String> res = new ArrayList<>();
        for (String w: words) {
            // 替换遍历
            char[] str = w.toCharArray();
            for (int i = 0; i < str.length; i++) {
                if (str[i] == separator) str[i] = ' ';
            }
            
            res.addAll(
                Arrays.stream(new String(str).split(" "))
                    .filter(x -> !x.equals(""))
                    .collect(Collectors.toList())
            );
        }
        
        return res;
        
    }
}
```

---

## [T2. 合并后数组中的最大元素](https://leetcode.cn/contest/weekly-contest-355/problems/largest-element-in-an-array-after-merge-operations/)

合并的前提是$num[i] \le nums[i + 1]$, 而且这个合并呈现区间性

可以寻找到一个贪心的最优解，即逆序遍历，只要后缀和大于等于前一个元素，即可以合并为一个更大后缀和。

```java
class Solution {
    public long maxArrayValue(int[] nums) {
        
        long ans = 0;
        long r = 0;
    
        for (int i = nums.length - 1; i >= 0; i--) {
            if (r >= nums[i]) {
                r += nums[i];
            } else {
                r = nums[i];
            }
            ans = Math.max(ans, r);
        }
        
        return ans;
        
    }
}
```

---
## [T3. 长度递增组的最大数目](https://leetcode.cn/contest/weekly-contest-355/problems/maximum-number-of-groups-with-increasing-length/)

好像一开始题目看错了，看成一定要用limits的次数，导致后续思路全偏了。

留着，待会来补上，先让我哭会

其实比赛中，用了二分的思路，但是普通的贪心思路好像是错的，因为没有考虑到构造交换，这也这题难的地方。

代码参考了小羊的做法，构造交换这种思路，真的是神来之笔，拜服.

```java
class Solution {
    public int maxIncreasingGroups(List<Integer> usageLimits) {
        Collections.sort(usageLimits);
        int ans = 0;
        long left = 0;
        for (int i = 0; i < usageLimits.size(); i++) {
            left += usageLimits.get(i);
            if (left >= ans + 1) {
                left -= ans + 1;
                ans++;
            }
        }
        return ans;
    }
}
```

---

## [T4. 树中可以形成回文的路径数](https://leetcode.cn/contest/weekly-contest-355/problems/count-paths-that-can-form-a-palindrome-in-a-tree/)

这题如果把 $\color{green}树结构去掉，换成数组$，是不是很熟悉的感觉？

如果是数组的话，那就前缀异或和，然后用map计数，线性遍历即可。

回忆下回文串的特性

- 要么字符全是偶数次
- 要么只有一个字符是奇数次，其他全是偶数次

因此把字符串转换为一个26字母向量的0-1状态串

---

那回到树结构，如何解呢？

关于异或树，有个非常重要的特性

就是对于树节点$i,j$，它的最近公共祖先$t$, 根节点$r$

其 ${path(x, y) = path(x, t) \oplus path(t, y) = path(x, r) \oplus path(r, y)}$ 恒成立


然后令状态$S(x)= path(r, x)$, 相当于x节点，其根节点到x节点的前缀异或和

$path(x, y)=S(x) \oplus S(y)$ 为回文串, 则满足以下条件（2选1）

- $S(x) == S(y)$
- $S(x) \oplus S(y) = 2的幂次$


这就是解这题的所有理论基础，和前提条件


```java
class Solution {

    int n;

    char[] str;
    List<Integer> parent;

    List<Integer> []g;

    TreeMap<Integer, Integer> hash = new TreeMap<>();

    void dfs(int u, int fa, int xor) {
        if (u != 0) {
            int p = str[u] - 'a';
            xor ^= (1 << p);
        }

        hash.merge(xor, 1, Integer::sum);

        for (int v: g[u]) {
            if (v == fa) continue;
            dfs(v, u, xor);
        }
    }

    public long countPalindromePaths(List<Integer> parent, String s) {
        this.n = parent.size();
        this.parent = parent;
        g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());
        str = s.toCharArray();

        for (int i = 1; i < parent.size(); i++) {
            g[parent.get(i)].add(i);
        }

        dfs(0, -1, 0);

        long ans1 = 0, ans2 = 0;
        for (var kv: hash.entrySet()) {
            int k = kv.getKey();
            int v = kv.getValue();

            long r1 = (long)v * (v - 1) / 2;
            ans1 += r1;

            for (int i = 0; i < 26; i++) {
                int k2 = k ^ (1 << i);
                ans2 += (long) v * hash.getOrDefault(k2, 0);
            }
        }

        return ans1 + ans2 / 2;
    }

}
```


---

# 写在最后

深山踏红叶，耳畔闻鹿鸣。我很喜欢枫叶，可惜，枫叶红时，总多离别。


![image.png](https://pic.leetcode.cn/1690090019-XEjLvY-image.png)





