# 第 367 场周赛 解题报告 | 珂学家 | 前后缀拆解 + 四叉树解法

---
# 前言


![image.png](https://pic.leetcode.cn/1697342638-HZfkKv-image.png)


你指尖舞动的电光，是我此生不灭的信仰，最强无敌超电磁炮，吾辈毕生必然超越次元，生死相随

---
# 整体评价

没想到赞助赛是手速场，非常的友好。

T4 补充 四叉树 解法。

---
## [T1. 找出满足差值条件的下标 I](https://leetcode.cn/contest/weekly-contest-367/problems/find-indices-with-index-and-value-difference-i/)

同T3一起讲解

---
## [T2. 最短且字典序最小的美丽子字符串](https://leetcode.cn/contest/weekly-contest-367/problems/shortest-and-lexicographically-smallest-beautiful-string/)

模拟即可，因为数据范围小

如果数据量大，则需要单独抽取1构建一个索引序列

```java
class Solution {
    public String shortestBeautifulSubstring(String s, int k) {
        char[] str = s.toCharArray();
        int n = str.length;
        
        String ans = null;
        for (int i = 0; i < n; i++) {
            int tmp = 0;
            int pos = -1;
            for (int j = i; j < n; j++) {
                if (str[j] == '1') tmp++;
                if (tmp == k) {
                    pos = j;
                    break;
                }
            }
            if (pos != -1) {
                String t = new String(str, i, pos - i + 1);
                if (ans == null) ans = t;
                else if (ans.length() > t.length()) ans = t;
                else if (ans.length() == t.length() && ans.compareTo(t) > 0) {
                    ans = t;
                }
            }
        }
        return ans == null ? "" : ans;
    }
}
```

时间复杂度 $O(n^2)$

---
## [T3. 找出满足差值条件的下标 II](https://leetcode.cn/contest/weekly-contest-367/problems/find-indices-with-index-and-value-difference-ii/)

本质就是维护滑窗

因为只需要求一个任意可行解，就取特殊解，即最大，最小解。

```java
class Solution {
    public int[] findIndices(int[] nums, int indexDifference, int valueDifference) {
        
        long min = Long.MAX_VALUE / 10;
        long max = Long.MIN_VALUE / 10;
        int maxIdx = -1, minIdx = -1;

        int n = nums.length;
        for (int i = 0; i < n; i++) {
            int v = nums[i];
            
            // 把indexDifference的元素更新
            if (i - indexDifference >= 0 && i - indexDifference < n) {
                if (max < nums[i - indexDifference]) {
                    max = nums[i - indexDifference];
                    maxIdx = i - indexDifference;
                }
                if (min > nums[i - indexDifference]) {
                    min = nums[i - indexDifference];
                    minIdx = i - indexDifference;
                }
            }
            
            if (v + valueDifference <= max) return new int[] {i, maxIdx};
            if (v - valueDifference >= min) return new int[] {i, minIdx};
        }

        return new int[] {-1, -1};

    }
}
```

时间复杂度为$O(n)$


---
## [T4. 构造乘积矩阵](https://leetcode.cn/contest/weekly-contest-367/problems/construct-product-matrix/)

第一眼感觉是逆元，但是逆元有个前提条件

$$ a * x = 1\ mod\ b $$
$$a,b互质，才能求解a的逆元x，否则有可能不存在$$

如果不是二维的话，那一维就特别简单，前后缀拆解就行

但这题，可以把二维矩阵拍平成一维数组，亦能满足需求

$$res[i][j]=pre[i*w+j-1] * suf[i*w+j+1]$$

所以这题，也就这个考点，二维降一维。




```java
class Solution {
    public int[][] constructProductMatrix(int[][] grid) {
        int h = grid.length, w = grid[0].length;
        long[] pre = new long[h * w];
        long[] suf = new long[h * w];
        
        // 计算前缀
        long mod = 12345;
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (i == 0 && j == 0) pre[0] = grid[0][0] % mod;
                else pre[i * w + j] = pre[i * w + j - 1] * grid[i][j] % mod;
            }
        }

        // 计算后缀        
        for (int i = h - 1; i >= 0; i--) {
            for (int j = w - 1; j >= 0; j--) {
                if (i == h - 1 && j == w - 1) suf[h * w - 1] = grid[h - 1][w - 1] % mod;
                else suf[i * w + j] = suf[i * w + j + 1] * grid[i][j] % mod;
            }
        }
        
        int[][] res = new int[h][w];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (i==0 && j == 0) {
                    res[0][0] = (int)suf[1];
                } else if (i == h - 1 && j == w - 1) {
                    res[h - 1][w - 1] = (int)pre[h * w - 2];
                } else {
                    res[i][j] = (int)(pre[i * w + j - 1] * suf[i * w + j + 1] % mod);
                }
            } 
        }
        
        return res;
        
    }
}
```

---

## 四叉树解法

这题也可以用四叉树来求解的

每个节点是一个矩形区域

![image.png](https://pic.leetcode.cn/1697349844-cSVPqF-image.png)


```java
class Solution {

    static class SquareTree {
        int ty, tx, by, bx;
        long val;

        SquareTree[] cs;

        public SquareTree(int[][] grid, int mod, int ty, int tx, int by, int bx) {
            this.ty = ty;
            this.tx = tx;
            this.by = by;
            this.bx = bx;

            if (ty == by && tx == bx) {
                this.val = grid[ty][tx] % mod;
                return;
            }

            int m1 = ty + (by - ty) / 2;
            int m2 = tx + (bx - tx) / 2;

            cs = new SquareTree[4];
            this.cs[0] = new SquareTree(grid, mod, ty, tx, m1, m2);
            if (m2 + 1 <= bx) {
                this.cs[1] = new SquareTree(grid, mod, ty, m2 + 1, m1, bx);
            }
            if (m1 + 1 <= by) {
                this.cs[2] = new SquareTree(grid, mod, m1 + 1, tx, by, m2);
            }
            if (m1 + 1 <= by && m2 + 1 <= bx) {
                this.cs[3] = new SquareTree(grid, mod, m1 + 1, m2 + 1, by, bx);
            }

            this.val = 1l;
            for (int i = 0; i < 4; i++) {
                if (this.cs[i] != null) this.val = this.val * this.cs[i].val % mod;
            }
        }

        long query(int y, int x, int mod) {
            if (isLeaf()) {
                return y == ty && x == tx ? 1 : this.val;
            } else {
                long r = 1l;
                for (int i = 0; i < 4; i++) {
                    if (cs[i] != null) {
                        // 包含则要继续递归下去
                        if (cs[i].contains(y, x)) r = r * cs[i].query(y, x, mod) % mod;
                        else r = r * cs[i].val % mod;
                    }
                }
                return r;
            }
        }

        // 是否是叶子节点
        boolean isLeaf() {
            return ty == by && tx == bx;
        }

        // 是否包含该点
        boolean contains(int y, int x) {
            return this.ty <= y && this.tx <= x && this.by >= y && this.bx >= x;
        }

    }

    public int[][] constructProductMatrix(int[][] grid) {
        int h = grid.length, w = grid[0].length;
        int mod = 12345;

        SquareTree tree = new SquareTree(grid, mod, 0, 0, h - 1, w - 1);

        int[][] res = new int[h][w];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                res[i][j] = (int)tree.query(i, j, mod);
            }
        }

        return res;

    }
}
```

---
# 写在最后


人的认知不不易于书对人的启示

![image.png](https://pic.leetcode.cn/1697342672-rWGuiP-image.png)
