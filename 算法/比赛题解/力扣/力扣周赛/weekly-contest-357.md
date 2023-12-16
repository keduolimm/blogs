# 第 357 场周赛 解题报告 | 珂学家 | 区间DP + 多源BFS/二分 + 多变量贪心


---

# 前言

![image.png](https://pic.leetcode.cn/1691298344-hhlmwa-image.png)

人生不过百年，何必太过计较。

---

# 整体评价

这场码量还是蛮大的，T2是经典的区间DP题，好像力扣很久很出这类题了，T3一眼多源BFS，然后二分check. T4是经典贪心题，但是怎么个贪法，很有考究。

算是一场质量非常高的比赛.

---

## [T1. 故障键盘](https://leetcode.cn/contest/weekly-contest-357/problems/faulty-keyboard/)

模拟题(也算API题), 字符$i$变成了指令，其他字符照旧

```java
class Solution {
    public String finalString(String s) {
    
        StringBuilder sb = new StringBuilder();
        for (char c: s.toCharArray()) {
            if (c == 'i') {
                sb.reverse();
            } else {
                sb.append(c);
            }
        }
        
        return sb.toString();
        
    }
}
```

---

## [T2. 判断是否能拆分数组](https://leetcode.cn/contest/weekly-contest-357/problems/check-if-it-is-possible-to-split-array/)

经典的区间DP题，这题比较适合记忆化搜索

这边写个正推的DP解法

就是从小到大枚举长度，然后区间求解

需要前缀和优化预处理

时间复杂度为$O(n^3)$

```java
class Solution {

    public boolean canSplitArray(List<Integer> nums, int m) {

        int n = nums.size();
        
        // 前缀和预处理
        int[] prev = new int[n + 1];
        for (int i = 0; i < n; i++) {
            prev[i + 1] = prev[i] + nums.get(i);
        }

            
        // DP初始化
        int[][] opt = new int[n][n];
        for (int i = 0; i < n; i++) {
            Arrays.fill(opt[i], Integer.MIN_VALUE / 10);
        }
        // 长度为1的区间
        for (int i = 0; i < n; i++) {
            opt[i][i] = 1;
        }

        // 枚举长度
        for (int i = 2; i <= n; i++) {
            // 枚举起点
            for (int j = 0; j + i <= n; j++) {
                // 整体为一块
                if (prev[j + i] - prev[j] >= m) {
                   opt[j][j + i - 1] = 1; 
                }

                // 枚举分割的中点
                for (int k = 0; k < i - 1; k++) {
                    boolean f1 = k == 0 || prev[j + k + 1] - prev[j] >= m;
                    boolean f2 = k == i - 2 || prev[j + i] - prev[j + k + 1] >= m;
                    // 保证分割的前提
                    if (f1 && f2) {
                        opt[j][j + i - 1] = Math.max(opt[j][j + i - 1], opt[j][j + k] + opt[j + k + 1][j + i - 1]);
                    }
                }
            }
        }

        return opt[0][n - 1] >= n;
    }

}
```
---

补充$O(n)$的解法

因为$n\ge 3$拆解过程中，必然会产生长度为2的子数组(需要保证其和$\ge m$)

反之，存在一个相邻元素和$\ge m$必然可以引导拆分为n个子数组的过程。

两者互为充要条件

对于$n=1,2$的数组 天然满足要求

```java
class Solution {

    public boolean canSplitArray(List<Integer> nums, int m) {
        return nums.size() <= 2 || 
                IntStream.range(0, nums.size() - 1)
                    .anyMatch(i -> nums.get(i) + nums.get(i + 1) >= m);
    }

}
```


---
## [T3. 找出最安全路径](https://leetcode.cn/contest/weekly-contest-357/problems/find-the-safest-path-in-a-grid/)

一眼多源BFS, 然后二分check

> 题意中的 曼哈顿 距离， 本质等价于 左右上下移动的代价和。

多源BFS属于预处理，对那个格子进行标记

二分纯粹就是惯性，感觉DP有点绕，因为路劲并非向右/向下限定。

就是码量有点大

```java
class Solution {

    int[][] dirs = {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    boolean check(List<List<Integer>> grid, int[][] path, int limit) {
        int h = grid.size(), w = grid.get(0).size();

        Deque<int[]> deq = new ArrayDeque<>();
        if (path[0][0] < limit || path[h - 1][w - 1] < limit) return false;

        boolean[][] used = new boolean[h][w];
        deq.offer(new int[] {0, 0});
        used[0][0] = true;

        while (!deq.isEmpty()) {
            int[] cur = deq.poll();
            int y = cur[0], x = cur[1];
            for (int i = 0; i < dirs.length; i++) {
                int y1 = y + dirs[i][0];
                int x1 = x + dirs[i][1];
                if (y1 >= 0 && y1 < h && x1 >= 0 && x1 < w) {
                    if (!used[y1][x1] && path[y1][x1] >= limit) {
                        deq.offer(new int[] {y1, x1});
                        used[y1][x1] = true;
                        if (y1 == h - 1 && x1 == w - 1) return true;
                    }
                }
            }
        }

        return used[h - 1][w - 1];
    }
    
    // 多源bfs预处理
    public int[][] mbfs(List<List<Integer>> grid) {
        int h = grid.size(), w = grid.get(0).size();

        // 预处理，使用多源BFS，对那个格子进行标定
        int inf = Integer.MAX_VALUE / 10;
        int[][] path = new int[h][w];
        for (int i = 0; i < h; i++) Arrays.fill(path[i], inf);
        Deque<int[]> deq = new ArrayDeque<>();
        for (int i = 0; i < h; i++) {
            List<Integer> row = grid.get(i);
            for (int j = 0; j < w; j++) {
                int v = row.get(j);
                if (v == 1) {
                    deq.offer(new int[] {i, j, 0});
                    path[i][j] = 0;
                }
            }
        }
        while (!deq.isEmpty()) {
            int[] cur = deq.poll();
            int y = cur[0], x= cur[1], v = cur[2];
            for (int i = 0; i < dirs.length; i++) {
                int y1 = y + dirs[i][0];
                int x1 = x + dirs[i][1];
                if (y1 >= 0 && y1 < h && x1 >= 0 && x1 < w) {
                    if (path[y1][x1] > v + 1) {
                        path[y1][x1] = v + 1;
                        deq.offer(new int[] {y1, x1, v + 1});
                    }
                }
            }
        }
        return path;
    }

    public int maximumSafenessFactor(List<List<Integer>> grid) {
        
        // 1. 多源bfs预处理
        int[][] path = mbfs(grid);
        
        // 2. 二分求解
        int l = 0, r = grid.size() + grid.get(0).size();
        while (l <= r) {
            int m = l + (r - l) / 2;
            if (check(grid, path, m)) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }

        return r;
    }

}
```

---

## [T4. 子序列最大优雅度](https://leetcode.cn/contest/weekly-contest-357/problems/maximum-elegance-of-a-k-length-subsequence/)


因为这题数据量有些大，感觉背包DP无法施展手脚。

那这题的求解思路是啥呢？

> 这题本质上 双变量 求整体最优化的问题？

以往的思路，基本都是固定一个，然后求另一个变量，能取得整体的最优解。

但是如果这边固定 category 的数量，感觉这个还是难以下手。

那在回归到贪心思路上, 这里是否存在偏序上

> 先按profit从大到小排序，再获取profit最大的前k项，
> 后续的项(n-k)如果要提升整体值，唯有通过增加category数目来优化。

这个结论非常的重要，也是这题的关键。

```java
class Solution {

    public long findMaximumElegance(int[][] items, int k) {
        
        int n = items.length;

        // 按profit进行排序
        Arrays.sort(items, Comparator.comparingInt(x -> -x[0]));

        // 先获取前k项
        long acc = 0;
        Set<Integer> hash = new HashSet<>();
        PriorityQueue<int[]> pp = new PriorityQueue<>(Comparator.comparing(x -> x[0]));
        for (int i = 0; i < k; i++) {
            int[] item = items[i];
            if (hash.contains(item[1])) {
                pp.offer(item);
            } 
            hash.add(item[1]);
            acc += item[0];
        }
    
        // 目前的值
        long ncat = hash.size();
        long ans = acc + ncat * ncat;

        // 后续n-k个项，尝试让种类变多
        for (int i = k; i < n; i++) {
            int[] cur = items[i];
            int p = cur[0], cat = cur[1];
            if (!pp.isEmpty() && !hash.contains(cat)) {
                long w = pp.poll()[0];
                
                // 这边不需要比较新值/旧值，单纯添加新类别
                ncat++;
                acc = acc - w + p;
                hash.add(cat); // 添加种类
                ans = Math.max(ans, acc + ncat * ncat);
            }
        }

        return  ans;

    }
}
```



---

# 写在最后

不要为了别人而改变自己，要做真正的自己。

![image.png](https://pic.leetcode.cn/1691298386-HKFwpx-image.png)


