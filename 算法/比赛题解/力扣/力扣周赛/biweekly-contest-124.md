

# 第 124 场双周赛 解题报告 | 珂学家 | 非常规区间合并

---
# 前言

---
# 整体评价
T4的dp解法没想到，走了一条"不归路", 这个区间合并解很特殊，它是带状态的，而且最终的正解也是基于WA的case，慢慢理清的。
真心不容易，太难了。

---
## [T1. 相同分数的最大操作数目 I](https://leetcode.cn/contest/biweekly-contest-124/problems/maximum-number-of-operations-with-the-same-score-i/)

思路: 模拟

```java []
class Solution {
    
    public int maxOperations(int[] nums) {
        int n = nums.length;
        int res = 1;
        for (int i = 2; i + 1 < n; i+= 2) {
            if (nums[i] + nums[i + 1] == nums[0] + nums[1]) {
                res++;
            } else {
                break;
            }
        }
        return res;
    }
    
}
```

---
## [T2. 进行操作使字符串为空](https://leetcode.cn/contest/biweekly-contest-124/problems/apply-operations-to-make-string-empty/)

思路: 模拟
感觉有点绕

```java []
class Solution {
    public String lastNonEmptyString(String s) {
        List<Integer> []g = new List[26];
        Arrays.setAll(g, x->new ArrayList<>());
        for (int i = 0; i < s.length(); i++) {
            int p = s.charAt(i) - 'a';
            g[p].add(i);
        }
        
        int mz = 0;
        for (int i = 0; i < 26; i++) {
            mz = Math.max(g[i].size(), mz);
        }
        
        List<int[]> lasts = new ArrayList<>();
        for (int i = 0; i < 26; i++) {
            if (g[i].size() == mz) {
                lasts.add(new int[] {i, g[i].get(mz - 1)});
            }
        }
        
        Collections.sort(lasts, Comparator.comparing(x -> x[1]));
        StringBuilder sb = new StringBuilder();
        for (int[] e: lasts) {
            sb.append((char)(e[0] + 'a'));
        }
        return sb.toString();
        
    }
}
```

---

## [T3. 相同分数的最大操作数目 II]()

思路: 枚举+区间DP

因为要求和相等，所以枚举最初的和，然后记忆化搜索一下就出来了

```java []
class Solution {
    
    int dfs(Integer[][] dp, int[] nums, int s, int e, int v) {
        if (s >= e) return 0;
        if (dp[s][e] != null) return dp[s][e];
        
        int res = 0;
        if (nums[s] + nums[e] == v) {
            int r = dfs(dp, nums, s + 1, e - 1, v);
            res = Math.max(res, r + 1);
        }
        if (nums[s] + nums[s + 1] == v) {
            int r = dfs(dp, nums, s + 2, e, v);
            res = Math.max(res, r + 1);
        }            
        if (nums[e - 1] + nums[e] == v) {
            int r = dfs(dp, nums, s, e - 2, v);
            res = Math.max(res, r + 1);
        }
        
        return dp[s][e] = res;
    }
    
    public int maxOperations(int[] nums) {
        int n = nums.length;
        int r1 = dfs(new Integer[n][n], nums, 1, n - 2, nums[0] + nums[n - 1]);
        int r2 = dfs(new Integer[n][n], nums, 2, n - 1, nums[0] + nums[1]);
        int r3 = dfs(new Integer[n][n], nums, 0, n - 3, nums[n - 2] + nums[n - 1]);
        
        return Math.max(r1, Math.max(r2, r3)) + 1;
    }
    
}
```

---
## [T4. 修改数组后最大化数组中的连续元素数目](https://leetcode.cn/contest/biweekly-contest-124/problems/maximize-consecutive-elements-in-an-array-after-modification/)

思路: 区间合并

但是这个区间合并很特别，是带状态的

```java []
class Solution {

    static class Segment {
        int start, end;
        int lastStart, full;
        public Segment(int start, int end, int lastStart, int full) {
            this.start = start;
            this.end = end;
            this.lastStart = lastStart;
            this.full = full;
        }
    }
    
    public int maxSelectedElements(int[] nums) {

        int n = nums.length;
        Arrays.sort(nums);

        List<Segment> segs = new ArrayList<>();
        int i = 0;
        while (i < n) {
            int flag = 0;
            int j = i + 1;
            while (j < n && nums[j - 1] + 1 >= nums[j]) {
                if (nums[j - 1] == nums[j]) {
                    flag = 1;
                }
                j++;
            }
            segs.add(new Segment(nums[i], nums[j - 1], nums[i], flag));
            i = j;
        }

        Segment pre = null;
        int res = 0;
        for (Segment seg: segs) {
            if (pre == null) {
                pre = new Segment(seg.start, seg.end, seg.start, seg.full);
            } else {
                if (pre.end + 2 == seg.start) {
                    if (pre.full == 1) {
                        pre = new Segment(pre.start, seg.end, seg.start, seg.full);
                    } else {
                        pre = new Segment(pre.lastStart + 1, seg.end, seg.start, seg.full);
                    }
                } else {
                    pre = new Segment(seg.start, seg.end, seg.start, seg.full);
                }
            }

            res = Math.max(res, pre.end - pre.start + 1);
            if (pre.full == 1) {
                res = Math.max(res, pre.end - pre.start + 2);
            }
        }
        return res;
    }

}
```

---
# 写在最后

