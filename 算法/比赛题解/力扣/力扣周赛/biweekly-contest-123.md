
# 第 123 场双周赛 解题报告 | 珂学家 | 二维偏序+单调队列优化

# 前言

![image.png](https://pic.leetcode.cn/1706977542-djsFrz-image.png)


执手看歌敲金钗，笑语落珠明眸睐。
忽然蝴蝶春风满，焉教冷镜瘦朱颜。


---
# 整体评价
T3是基于map的前缀和的变形题，T4是二维偏序的一道应用题。

题外话，力扣还是实现N久之前的承诺了，命名权奖励，赞一个。

---

## [T1. 三角形类型 II](https://leetcode.cn/contest/biweekly-contest-123/problems/type-of-triangle-ii/)

思路: 模拟

```java []
class Solution {
    public String triangleType(int[] nums) {
        // 先判合法性
        Arrays.sort(nums);
        if (nums[0] + nums[1] <= nums[2]) return "none";
        
        if (nums[0] == nums[1] && nums[1] == nums[2]) {
            return "equilateral";
        } else if (nums[0] == nums[1] || nums[1] == nums[2]) {
            return "isosceles";
        } else {
            return "scalene";
        }
    }
}
```

---

## [T2. 人员站位的方案数 I](https://leetcode.cn/contest/biweekly-contest-123/problems/find-the-number-of-ways-to-place-people-i/)

和T4一起讲

---
## [T3. 最大好子数组和](https://leetcode.cn/contest/biweekly-contest-123/problems/maximum-good-subarray-sum/)

思路: 基于map的前缀和应用

这边需要以值作为key, value为最小的前缀和(需向前偏移一位)

更新的时候，需要分类讨论，v为当前值

- $v - k$
- $v + k$


```java []
class Solution {
    public long maximumSubarraySum(int[] nums, int k) {
        long inf = Long.MIN_VALUE / 10;
        long res = inf;

        // 维护最小的前缀和
        Map<Long, Long> minMap = new HashMap<>();

        long acc = 0;
        for (int i = 0; i < nums.length; i++) {
            long v = nums[i];
            acc += v;

            if (minMap.containsKey(v - k)) {
                res = Math.max(acc - minMap.get(v - k), res);
            }
            if (minMap.containsKey(v + k)) {
                res = Math.max(acc - minMap.get(v + k), res);
            }

            // 更新
            if (!minMap.containsKey(v) || acc - v < minMap.get(v)) {
                minMap.put(v, acc - v);
            }
        }

        return res == inf ? 0 : res;
    }

}
```

---

## [T4. 人员站位的方案数 II](https://leetcode.cn/contest/biweekly-contest-123/problems/find-the-number-of-ways-to-place-people-ii/)

思路: 二维偏序 + 枚举

对于偏序题，一般先固定一个维度

1. 先按x坐标从小到大排序，
2. 再按照y坐标从大到小排序

因为题目指定左上角，右下角

然后枚举左右端点，check是否满足需求即可。

在枚举的过程中，可以引入

$$单调队列优化$$

实际上只要维护最接近左端点y坐标(严格小于等于)的单变量即可, 递增状态

这样整个时间复杂度可以降为

- 排序$O(nlogn)$
- 枚举左右端点$O(n^2)$

最终为$O(n^2)$

```java []
class Solution {
    public int numberOfPairs(int[][] points) {
        // 按x从小到大，按y从大到小
        Arrays.sort(points, Comparator.comparingInt((int[] p) -> p[0]).thenComparingInt(p -> -p[1]));
        int res = 0;
        int n = points.length;
        for (int i = 0; i < n; i++) {
            // 维护最接近左端点y值的值(严格小于等于)
            int nearest = Integer.MIN_VALUE;
            for (int j = i + 1; j < n; j++) {
                if (points[j][1] <= points[i][1]) {
                    if (points[j][1] > nearest) {
                        res++;
                        nearest = points[j][1];
                    }
                }
            }
        }
        return res;
    }
}
```

---

- 离散化+二维前缀和 (补充)

这个解法应该更加的直观

```java []
class Solution {

    // 离散化
    Map<Integer, Integer> discrete(List<Integer> ps) {
        TreeSet<Integer> range = new TreeSet<>(ps);
        Map<Integer, Integer> ids = new HashMap<>();
        int ptr = 0;
        for (var k: range) {
            ids.put(k, ptr++);
        }
        return ids;
    }

    public int numberOfPairs(int[][] points) {
        int n = points.length;
        int res = 0;

        Map<Integer, Integer> xs = discrete(Arrays.stream(points).map(p -> p[0]).collect(Collectors.toList()));
        Map<Integer, Integer> ys = discrete(Arrays.stream(points).map(p -> p[1]).collect(Collectors.toList()));

        int h = ys.size(), w = xs.size();
        int[][] area = new int[h][w];
        for (int[] p: points) {
            area[ys.get(p[1])][xs.get(p[0])] = 1;
        }
        int[][] pre = new int[h + 1][w + 1];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                pre[i + 1][j + 1] = pre[i + 1][j] + pre[i][j + 1] - pre[i][j] + area[i][j];
            }
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (i == j) continue;
                if (points[i][0] <= points[j][0] && points[i][1] >= points[j][1]) {

                    int ty = ys.get(points[i][1]), by = ys.get(points[j][1]);
                    int tx = xs.get(points[j][0]), bx = xs.get(points[i][0]);

                    int s = pre[ty + 1][tx + 1] - pre[ty + 1][bx] - pre[by][tx + 1] + pre[by][bx];
                    if (s == 2) {
                        res ++;
                    }
                }
            }
        }

        return res;
    }
}

```

---

# 写在最后

![image.png](https://pic.leetcode.cn/1706977691-DPCeZq-image.png)
