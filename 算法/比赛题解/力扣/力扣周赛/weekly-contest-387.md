

# 第 387 场周赛 解题报告 | 珂学家 | 离散化&树状数组 + 模拟场

---
# 前言

![image.png](https://pic.leetcode.cn/1709450005-maIqiR-image.png)


---

# 整体评价

手速场+模拟场，思路和解法都蛮直接的。

所以搞点活

- 如果T2，如果不固定左上角，批量查询某个点为左上角，求满足总和$\le k$的子矩阵个数

- 如果T2，如果不固定左上角，求总和$\le k$的子矩阵个数

- 如果T3, 数值不局限于0,1,2, 求最小操作数

---

## [A. 将元素分配到两个数组中 I](https://leetcode.cn/contest/weekly-contest-387/problems/distribute-elements-into-two-arrays-i/)

思路: 模拟

模拟即可，没啥可说的。

```java []
class Solution {
    public int[] resultArray(int[] nums) {
        List<Integer> r1 = new ArrayList<>(List.of(nums[0]));
        List<Integer> r2 = new ArrayList<>(List.of(nums[1]));

        for (int i = 2; i < nums.length; i++) {
            if (r1.get(r1.size() - 1) > r2.get(r2.size() - 1)) {
                r1.add(nums[i]);
            } else {
                r2.add(nums[i]);
            }
        }
        
        r1.addAll(r2);
        return r1.stream().mapToInt(Integer::valueOf).toArray();
    }
}
```

---
## [B. 元素和小于等于 k 的子矩阵的数目](https://leetcode.cn/contest/weekly-contest-387/problems/count-submatrices-with-top-left-element-and-sum-less-than-k/)


思路: 二维前缀和 + 枚举

因为固定左上角，所以子矩阵的个数为$n * m$

前缀和预处理, $O(n * m)$

枚举子矩阵为, $O(n * m)$

```java []
class Solution {
    public int countSubmatrices(int[][] grid, int k) {
        int h = grid.length, w = grid[0].length;
        long[][] pre = new long[h + 1][w + 1];
        
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                pre[i + 1][j + 1] = pre[i + 1][j] + pre[i][j + 1] - pre[i][j] + grid[i][j];
            }
        }
        
        int res = 0;
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (pre[i + 1][j + 1] <= k) {
                    res ++;
                }
            }
        }
        
        return res;
    }
}
```
---

### 思考:

如果左上角并不固定，而且以任意点出发，求满足要求的子矩阵数？ 而且这个查询量不小？

那面对这个问题，该如何求解呢?

感觉一次查询，可以从 $O(n * m) 优化为 O(n+m)$，就是从右上点出发，逐渐收敛到左下。


---

## [C. 在矩阵上写出字母 Y 所需的最少操作次数](https://leetcode.cn/contest/weekly-contest-387/problems/minimum-operations-to-write-the-letter-y-on-a-grid/)


思路: 模拟 + 枚举组合

唯一可以增加难度的是，不限定数值范围

不过这也才基本的nlargest问题

```java []
class Solution {

    boolean isJudge(int y, int x, int n) {
        if (y == x && y <= n / 2) {
            return true;
        }
        if (y + x == n - 1 && y <= n / 2) {
            return true;
        }
        if (y >= n / 2 && x == n / 2) {
            return true;
        }
        return false;
    }

    public int minimumOperationsToWriteY(int[][] grid) {
        int n = grid.length;
        int[] ys = new int[3];
        int[] nys = new int[3];

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                int id = grid[i][j];
                if (isJudge(i, j, n)) {
                    ys[id]++;
                } else {
                    nys[id]++;
                }
            }
        }

        // 枚举即可
        int res = n * n;
        int totYs = n/2 + n/2 + n/2 + 1;
        int totNys = n * n - totYs;
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (i != j) {
                    res = Math.min(res, (totYs - ys[i]) + (totNys - nys[j]));
                }
            }
        }

        return res;
    }

}
```


---

## [D. 将元素分配到两个数组中 II](https://leetcode.cn/contest/weekly-contest-387/problems/distribute-elements-into-two-arrays-ii/)

思路：离散化 + 树状数组

板子题，而且非常的直接

```java []
class Solution {

    static class BIT {
        int n;
        int[] arr;
        public BIT(int n) {
            this.n =n;
            this.arr = new int[n + 1];
        }
        int query(int p) {
            int res = 0;
            while (p > 0) {
                res += arr[p];
                p -= p & -p;
            }
            return res;
        }
        void update(int p, int d) {
            while (p <= n) {
                arr[p] += d;
                p += p & -p;
            }
        }
    }

    public int[] resultArray(int[] nums) {
        List<Integer> arr1 = new ArrayList<>(List.of(nums[0]));
        List<Integer> arr2 = new ArrayList<>(List.of(nums[1]));
        
        // 离散化过程
        TreeSet<Integer> ts = new TreeSet<>();
        for (int v: nums) ts.add(v);
        int ptr = 1;
        Map<Integer, Integer> idMap = new HashMap<>();
        for (var k: ts) {
            idMap.put(k, ptr++);
        }

        // 树状数组模拟过程
        BIT bit1 = new BIT(ptr);
        BIT bit2 = new BIT(ptr);

        bit1.update(idMap.get(nums[0]), 1);
        bit2.update(idMap.get(nums[1]), 1);
        for (int i = 2; i < nums.length; i++) {
            int v = nums[i];
            Integer k = idMap.get(v);

            int cnt1 = bit1.query(ptr) - bit1.query(k);
            int cnt2 = bit2.query(ptr) - bit2.query(k);
            if (cnt1 > cnt2 || (cnt1 == cnt2 && arr2.size() >= arr1.size())) {
                arr1.add(v);
                bit1.update(k, 1);
            } else {
                arr2.add(v);
                bit2.update(k, 1);
            } 
        }
        
        arr1.addAll(arr2);
        return arr1.stream().mapToInt(Integer::valueOf).toArray();
    }
}
```

---
# 写在最后
![image.png](https://pic.leetcode.cn/1709449974-TlOBCG-image.png)



