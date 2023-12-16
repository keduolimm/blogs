# 第 354 场周赛 解题报告 | 珂学家 | 字典树/AC自动机 + 差分 + 前后缀拆解


---
# 前言

![image.png](https://pic.leetcode.cn/1689478449-vRlEUN-image.png)

与我而言，不变也是一种美好，哎呀，可尘世那是如此不变之物。

---

# 整体评价

感觉涉及的知识点还是蛮广的，算综合场，而且整体码量都不大。但是感觉T4有点水，如果没有限制10，好像也没啥好办法，放大的话，估计可以卡常.

---

## [T1. 特殊元素平方和](https://leetcode.cn/problems/sum-of-squares-of-special-elements/)

本质上，就是, 签到题

> $长度 \% 索引 == 0$

```java
class Solution {
    public int sumOfSquares(int[] nums) {
        int ans = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums.length % (i + 1) == 0) {
                ans += nums[i] * nums[i];
            }
        }
        return ans;
    }
}
```

---

## [T2. 数组的最大美丽值](https://leetcode.cn/problems/maximum-beauty-of-an-array-after-applying-operation/)

非常经典的差分应用题, 注意该题是**子序列，不是子数组**

因为$nums[i], k$的范围在$1e5$, 因此可以引入偏移量就行

对于每个$nums[i]$, 可以对应一个值域区间$[nums[i] - k, nums[i] + k]$

因此差分的对象，其实是值域的概念

枚举的话，也是从值域的角度出发

```java
class Solution {
    
    public int maximumBeauty(int[] nums, int k) {
    
        int offset = k;
        
        int mz = Arrays.stream(nums).max().getAsInt();
        
        int[] diff = new int[offset + k + mz + 2];
        
        for (int i = 0; i < nums.length; i++) {
            diff[nums[i] - k + offset]++;
            diff[nums[i] + k + 1 + offset]--;
        }
        
        int ans = 0;
        int acc = 0;
        for (int i = 0; i < diff.length; i++) {
            
            acc += diff[i];
            ans = Math.max(ans, acc);
        }
        
        return ans;
        
    }
    
}
```
---

### 方法二：双指针

因为k固定， 按nums值进行排序，然后使用双指针技巧

```java
class Solution {
    
    public int maximumBeauty(int[] nums, int k) {
        int ans = 0;
        int n = nums.length;
        Arrays.sort(nums);
        int j = 0;
        for (int i = 0; i < n; i++) {
            while (j < i && nums[j] + k < nums[i] - k) {
                j++;
            }
            ans = Math.max(ans, i - j + 1);
        }
        return ans;
    }

}
```

### 题外话

如果这题不是子序列，而是子数组，那这题又改如何解呢？

其实这样的话，就和前几周周赛的题重复了。

思路也是滑窗双指针，遍历枚举右端点

那这个滑窗如何维护呢？

- 维护单调栈(最大&最小)
- 利用TreeMap维护区间内的端点最值

---

## [T3. 合法分割的最小下标](https://leetcode.cn/problems/minimum-index-of-a-valid-split/)

前后缀拆解的思路

预处理左侧，生成一个主导数组，同时预处理右侧，生成一个主导数组

枚举边界，判定是否满足条件: 左右两侧主导元素相等

这边的主导定义：**必须超过一半**

这也让题目简单了很多

```java
class Solution {

    public int minimumIndex(List<Integer> nums) {

        int n = nums.size();
        int[] prev = new int[n];
        int[] sub = new int[n];

        int hou1 = 0, cnt1 = 0;
        Map<Integer, Integer> hash1 = new HashMap<>();
        for (int i = 0; i < n; i++) {
            int u = nums.get(i);
            hash1.merge(u, 1, Integer::sum);

            int cnt = hash1.get(u);
            if (cnt > cnt1) {
                cnt1 = cnt;
                hou1 = u;
            }
            
            if (cnt1 > (i + 1) / 2) {
                prev[i] = hou1;
            }
        }

        int hou2 = 0, cnt2 = 0;
        Map<Integer, Integer> hash2 = new HashMap<>();
        for (int i = n - 1; i >= 0; i--) {
            int u = nums.get(i);
            hash2.merge(u, 1, Integer::sum);
            
            int cnt = hash2.get(u);
            if (cnt > cnt2) {
                cnt2 = cnt;
                hou2 = u;
            }
            
            if (cnt2 > (n - 1 - i + 1) / 2) {
                sub[i] = hou2;
            }
        }

        for (int i = 0; i < n - 1; i++) {
            if (prev[i] > 0 && sub[i + 1] > 0) {
                if (prev[i] == sub[i + 1]) {
                    return i;
                }
            }
        }

        return -1;

    }

}
```

---

## [T4. 最长合法子字符串的长度](https://leetcode.cn/problems/length-of-the-longest-valid-substring/)

这题写个字典树吧

思路其实还是滑窗，遍历枚举右端点，在原先合法区间$[l, r]$基础上加上右端点字符，然后试图匹配寻找 **非法的单词**

因为从右侧开始遍历，所以构建字典树的时候，需要预处理逆转

说说字典树的好处吧

- 可以快速剪枝，因为公共前缀/后缀
- 更节省空间，因为前缀节点复用

题外话：

那这题，如果没限制 $\color{red}10$ 的大小，字典树是否依旧能胜任？

感觉可能不行，可以构造极端case

> $'aaaa...aaaaaa', bans=['aaaaa....aaa']$

```java


class Solution {

    static class Trie {
        Trie[] cs = new Trie[26];
        boolean tag;
        public static void addWord(Trie root, String s) {
            Trie cur = root;
            for (char c: s.toCharArray()) {
                int p = c - 'a';
                if (cur.cs[p] == null) {
                    cur.cs[p] = new Trie();
                }
                cur = cur.cs[p];
            }
            cur.tag = true;
        }

        public static int search(Trie root, char[] str, int idx, int head) {
            Trie cur = root;
            for (int i = idx; i >= head; i--) {
                int p = str[i] - 'a';
                if (cur.cs[p] == null) return -1;
                cur = cur.cs[p];
                if (cur.tag) {
                    return i + 1;
                }
            }
            return head;
        }

    }

    public int longestValidSubstring(String word, List<String> forbidden) {

        // 构建字典树
        Trie root = new Trie();
        for (String s: forbidden) {
            Trie.addWord(root, new StringBuilder(s).reverse().toString());
        }

        int ans = 0;
        char[] str = word.toCharArray();
        int j = 0;

        for (int i = 0; i < str.length; i++) {
            int last = Trie.search(root, str, i, j);
            j = Math.max(j, last);
            ans = Math.max(ans, i - j + 1);
        }

        return ans;
    }

}

```

---

### 方法二：AC自动机

踩了一些坑，和多模板匹配处理有些不一样的地方

比如字典为 $["aabaa", "baaa", "aaa"], 字符串为"aabaaaxx", 需要处理字典中单词本身包含的关系$

```java
class Solution {

    static class Trie {

        Trie[] cs = new Trie[26];
        Trie fail;
        int depth;

        boolean tag;

        public static void addWord(Trie root, String s) {
            Trie cur = root;
            char[] str = s.toCharArray();
            for (int i = 0; i < str.length; i++) {
                int p = str[i] - 'a';
                if (cur.cs[p] == null) {
                    cur.cs[p] = new Trie();
                }
                cur = cur.cs[p];
                cur.depth = i;
            }
            cur.tag = true;
        }

        public static void getFail(Trie root) {

            Deque<Trie> deq = new LinkedList<>();
            deq.offer(root);

            while (!deq.isEmpty()) {
                Trie cur = deq.poll();
                for (int i = 0; i < 26; i++) {
                    if (cur.cs[i] != null) {
                        cur.cs[i].fail = root;

                        Trie fa = cur.fail;
                        while (fa != null) {
                            if (fa.cs[i] != null) {
                                cur.cs[i].fail = fa.cs[i];
                                break;
                            }
                            fa = fa.fail;
                        }

                        // 需要这个, 很重要，应对 ["baaa", "aaa"]这类
                        if (cur.cs[i].fail != root) {
                            if (cur.cs[i].fail.tag) {
                                cur.cs[i].tag = true;
                                cur.cs[i].depth = cur.cs[i].fail.depth;
                            }
                        }

                        deq.offer(cur.cs[i]);
                    }
                }
            }

        }

        public static void search(Trie root, char[] str, int[] dp) {
            Trie cur = root;
            for (int i = 0; i < str.length; i++) {
                // 传递
                dp[i] = (i > 0) ? dp[i - 1] : dp[0];

                int p = str[i] - 'a';
                while (cur != null && cur.cs[p] == null) {
                    cur = cur.fail;
                }

                cur = (cur == null) ? root : cur.cs[p];

                while (cur != root) {
                    if (cur.tag) {
                        // 做处理
                        // 然后进行更新操作
                        dp[i] = Math.max(dp[i], i - cur.depth + 1);
                        cur = cur.fail;
                    } else {
                        break;
                    }
                }
            }

        }

    }

    public int longestValidSubstring(String word, List<String> forbidden) {

        // 构建字典树
        Trie root = new Trie();
        for (String s: forbidden) {
            Trie.addWord(root, s);
        }
        Trie.getFail(root);

        int ans = 0;
        char[] str = word.toCharArray();
        int[] dp = new int[str.length];

        Trie.search(root, str, dp);
        for (int i = 0; i < str.length; i++) {
            ans = Math.max(ans, i - dp[i] + 1);
        }
        return ans;
    }

}
```


---

# 写在最后

我的神明，就托付给你了。

![image.png](https://pic.leetcode.cn/1689479135-jIKbfG-image.png)




