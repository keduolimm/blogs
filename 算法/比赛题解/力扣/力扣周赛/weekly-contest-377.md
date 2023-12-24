

# 第 377 场周赛 解题报告 | 珂学家 | Floyd + 划分型DP

---

# 前言

![image.png](https://pic.leetcode.cn/1703393779-eRlzFx-1703393724-CRBbTs-image.png)


---

# 整体评价

天崩局，压哨绝杀，感谢天，感谢地，T_T.

感觉被T2玩惨了，T3和T4很像，无非一个贪心，一个是划分型DP，但是都需要基于floyd预处理。

---

## [T1. 最小数字游戏](https://leetcode.cn/contest/weekly-contest-377/problems/minimum-number-game/)

思路： 模拟

排序/最小堆，模拟即可

```java []
class Solution {
    public int[] numberGame(int[] nums) {
        Arrays.sort(nums);
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < nums.length; i += 2) {
            res.add(nums[i + 1]);
            res.add(nums[i]);
        }
        return res.stream().mapToInt(Integer::valueOf).toArray();
    }
}
```

---
## [T2. 移除栅栏得到的正方形田地的最大面积](https://leetcode.cn/contest/weekly-contest-377/problems/maximum-square-area-by-removing-fences-from-a-field/)

思路：彼此独立 + 枚举hash

这类题，大概率往横纵坐标彼此独立的角度去思考。

可以计算求的横坐标/纵坐标的可能取值，求交集的最大值即可。

这题感觉卡常，set必须得hashset, treeset会tle

```java []
class Solution {
    Set<Integer> convert(int n, int[] arr) {
        Arrays.sort(arr);

        Set<Integer> res = new HashSet<Integer>();
        res.add(n - 1);
        for (int i = 0; i < arr.length; i++) {
            res.add(arr[i] - 1);
            for (int j = 0; j < i; j++) {
                res.add(arr[i] - arr[j]);
            }
            res.add(n - arr[i]);
        }
        return res;
    }

    public int maximizeSquareArea(int m, int n, int[] hFences, int[] vFences) {
        long mod = (long)1e9 + 7;
        Set<Integer> rows = convert(m, hFences);
        Set<Integer> cols = convert(n, vFences);

        // 逆序遍历
        List<Integer> lists = rows.stream().sorted().toList();
        for (int i = lists.size() - 1; i >= 0; i--) {
            int v = lists.get(i);
            if (cols.contains(v)) {
                return (int)((long)v * v % mod);
            }
        }

        return -1;
    }
}
```

---

## [T3. 转换字符串的最小成本 I](https://leetcode.cn/contest/weekly-contest-377/problems/minimum-cost-to-convert-string-i/)

思路：贪心

但是需要Floyd预处理，因为

$$A到B的最小代, 可能不是A->B最短，有可能A->C->B最短 $$

可能还需要注意，存在重边

```java []
class Solution {
    public long minimumCost(String source, String target, char[] original, char[] changed, int[] cost) {

        int inf = Integer.MAX_VALUE / 10;
        int[][] path = new int[26][26];
        for (int i = 0; i < 26; i++) {
            Arrays.fill(path[i], inf);
            path[i][i] = 0;
        }

        for (int i = 0; i < original.length; i++) {
            int s = original[i] - 'a';
            int e = changed[i] - 'a';
            path[s][e] = Math.min(path[s][e], cost[i]);
        }

        for (int k = 0; k < 26; k++) {
            for (int i = 0; i < 26; i++) {
                for (int j = 0; j < 26; j++) {
                    if (path[i][j] > path[i][k] + path[k][j]) {
                        path[i][j] = path[i][k] + path[k][j];
                    }
                }
            }
        }

        long res = 0;
        char[] bs = source.toCharArray();
        char[] ts = target.toCharArray();
        for (int i = 0; i < bs.length; i++) {
            int p1 = bs[i] - 'a', p2 = ts[i] - 'a';
            if (path[p1][p2] == inf) {
                return -1;
            }
            res += path[p1][p2];
        }
        return res;

    }
}

```

---

## [T4. 转换字符串的最小成本 II](https://leetcode.cn/contest/weekly-contest-377/problems/minimum-cost-to-convert-string-ii/)

思路：floyd + 划分型DP

把字符串视为节点，floyd预处理得到节点之间的转化代价，以为边稀疏，所以可以稍微优化下。

这题的关键是，两个描述的条件

$$ 操作的字符串没有重叠，满足划分型DP $$

令 opt[i] 为以i结尾的转换最小代价

用刷表法转移

- $opt[i + 1] = min(opt[i+1], opt[i]),  source[i+1] == target[i+1]$
- $opt[i + w] = min(opt[i+w], opt[i] + cost[source[i+1:i+w+1]][target[i+1:i+w+1]])$


```java []
class Solution {

    long inf = Long.MAX_VALUE / 10;
    Map<String, Integer> idMap = new HashMap<>();
    long[][] path;

    void init(String[] original, String[] changed, int[] cost) {
       // 离散化, str -> id
       TreeSet<String> ts = new TreeSet<>();
       for (String w: original) ts.add(w);
       for (String w: changed) ts.add(w);

       int ptr = 0;
       for (var k: ts) {
           idMap.put(k, ptr);
           ptr++;
       }

       path = new long[ptr][ptr];
       for (int i = 0; i < ptr; i++) {
           Arrays.fill(path[i], inf);
           path[i][i] = 0;
       }

       for (int i = 0; i < original.length; i++) {
           int t1 = idMap.get(original[i]);
           int t2 = idMap.get(changed[i]);
           path[t1][t2] = Math.min(path[t1][t2], cost[i]);
       }

       // floyd
       for (int k = 0; k < ptr; k++) {
           for (int i = 0; i < ptr; i++) {
               // 稀疏表，大剪枝
               if (path[i][k] == inf) continue;
               for (int j = 0; j < ptr; j++) {
                   if (path[i][j] > path[i][k] + path[k][j]) {
                       path[i][j] = path[i][k] + path[k][j];
                   }
               }
           }
       }
    }

    public long minimumCost(String source, String target, String[] original, String[] changed, int[] cost) {

        // 对长度去重, 减少枚举次数
        TreeSet<Integer> ts = new TreeSet<>();
        for (String s: original) ts.add(s.length());
        for (String s: changed) ts.add(s.length());
        List<Integer> lens = ts.stream().toList();

        
        init(original, changed, cost);

        int n = source.length();
        char[] str1 = source.toCharArray();
        char[] str2 = target.toCharArray();

        // dp核心逻辑
        long[] dp = new long[n + 1];
        Arrays.fill(dp, inf);
        dp[0] = 0;
        for (int i = 0; i < n; i++) {
            if (str1[i] == str2[i]) {
                dp[i + 1] = Math.min(dp[i + 1], dp[i]);
            }

            // 这部分有优化空间
            for (int j = 0; j < lens.size(); j++) {
                int w = lens.get(j);
                if (w > n - i) break;
                String s = new String(str1, i, w);
                String t = new String(str2, i, w);
                if (idMap.containsKey(s) && idMap.containsKey(t)) {
                    dp[i + t.length()] = Math.min(dp[i + t.length()], dp[i] + path[idMap.get(s)][idMap.get(t)]);
                }
            }
        }

        return dp[n] == inf ? -1 : dp[n];
    }
}
```

---

# 写在最后


![image.png](https://pic.leetcode.cn/1703393779-cQU＊＊＊T-1703393705-sZfIes-image.png)



