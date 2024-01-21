
# 第 122 场双周赛 解题报告 | 珂学家 | 脑筋急转弯 + 滑窗&反悔堆


---

# 前言

![image.png](https://pic.leetcode.cn/1705770092-vKOBAi-image.png)

---

# 整体评价

倒开差点崩盘，T4这个反悔堆写吐了，T3往众数上去猜了，幸好case良心。

---

## [T1. 将数组分成最小总代价的子数组 I](https://leetcode.cn/contest/biweekly-contest-122/problems/divide-an-array-into-subarrays-with-minimum-cost-i/)

思路: 取 nums[1:] 的最小2个值

可以部分排序，这样更快捷

```java []
class Solution {
    public int minimumCost(int[] nums) {
        Arrays.sort(nums, 1, nums.length);
        return nums[0] + nums[1] + nums[2];
    }
}
```

---

## [T2. 判断一个数组是否可以变为有序](https://leetcode.cn/contest/biweekly-contest-122/problems/find-if-array-can-be-sorted/)

思路: 分组分段排序

把连续同构子数组排序(同构即1的个数相同)

最后验证一把即可

```java []
class Solution {
    public boolean canSortArray(int[] nums) {
        // 分组分段排序
        int n = nums.length;
        int i = 0;
        while (i < n) {
            int j = i + 1;
            while (j < n && Integer.bitCount(nums[j - 1]) == Integer.bitCount(nums[j])) {
                j++;
            }
            Arrays.sort(nums, i, j);
            i = j;
        }
        
        // 最后验证下
        for (i = 0; i < n - 1; i++) {
            if (nums[i] > nums[i + 1]) return false;
        }
        return true;
    }
}
```

---

## [T3. 通过操作使数组长度最小](https://leetcode.cn/contest/biweekly-contest-122/problems/minimize-length-of-array-using-operations/)

思路: 找规律, 从最小数切入

可以发现:

$$小数可以剔除大数$$

比如$a,b两数(a\lt b), 则 a\ mod\ b = a, 相当于单独剔除b, 保留小数a$

$$那大数可以剔除小数呢？$$

- $a,b(a\lt b), a\ mod\ b \ne 0, 则构建了更小的数c(c\lt a)$
- $a,b(a\lt b), a\ mod\ b = 0, 则不行$

令最小值为v，个数为m

如果存在一个数，其余最小数不为0，则结果为1

如果不存在这样的数，则结果为$(m+1)/2$

```java []
class Solution {
    public int minimumArrayLength(int[] nums) {
        if (nums.length <= 2) return 1;
        
        int minV = Arrays.stream(nums).min().getAsInt();
        
        int minNum = 0;
        for (int v: nums) {
            // 存在一个数可以构造更小的数
            if (v % minV != 0) return 1;
            else if (v == minV) minNum++;
        }
        
        return (minNum + 1) / 2;
    }
}
```

---

## [T4. 将数组分成最小总代价的子数组 II](https://leetcode.cn/contest/biweekly-contest-122/problems/divide-an-array-into-subarrays-with-minimum-cost-ii/)

思路: 反悔堆

划分型DP只是幌子，核心是

$$滑窗区间(dist)内的最小k-1个数之和$$

这个过程中，有旧数据的滑出，有新数据的滑入，还有实时更新k-1个最小和。

除了滑窗的框架，还需要额外引入

$$对顶堆$$

即原先k-1最小的集合元素，因窗口挪动，被更小的新值替换(非窗口有效范围原因)



```java []
class Solution {

    public long minimumCost(int[] nums, int k, int dist) {
        long inf = Long.MAX_VALUE;
        long ans = inf;

        long res = nums[0];
        int n = nums.length;
        int cnt = 1;

        // 两者构建对顶堆
        // 候选堆(最小堆)
        PriorityQueue<int[]> minPq = new PriorityQueue<>(Comparator.comparing(x -> x[0]));
        // k-1集合堆(最大堆)
        PriorityQueue<int[]> maxPq = new PriorityQueue<>(Comparator.comparing(x -> -x[0]));

        for (int i = 1; i <= dist + 1; i++) {
            minPq.offer(new int[] {nums[i], i});
        }

        boolean[] vis = new boolean[n];
        while (cnt < k) {
            int[] cur = minPq.poll();
            res += cur[0];
            vis[cur[1]] = true;
            maxPq.offer(new int[] {cur[0], cur[1]});
            cnt++;
        }
        ans = Math.min(ans, res);

        for (int i = 1; i + 1 + dist < n; i++) {
            // 滑窗
            if (vis[i] == true) {
                cnt--;
                res -= nums[i];
            }

            minPq.offer(new int[] {nums[i + 1 + dist], i + 1 + dist});
            // 保证k-1大小
            while (cnt < k && !minPq.isEmpty()) {
                int[] cur = minPq.poll();
                // 惰性删除
                if (cur[1] <= i) continue;
                vis[cur[1]] = true;
                res += cur[0];
                cnt++;
                maxPq.offer(cur);
            }

            while (!maxPq.isEmpty() && !minPq.isEmpty()) {
                // 惰性删除
                int[] cur2 = maxPq.peek();
                if (cur2[1] <= i) {
                    maxPq.poll();
                    continue;
                }
                // 惰性删除
                int[] cur1 = minPq.peek();
                if (cur1[1] <= i) {
                    minPq.poll();
                    continue;
                }

                if (cur1[0] >= cur2[0]) break;

                // 反悔核心逻辑
                res = res + cur1[0] - cur2[0];
                vis[cur1[1]] = true;
                vis[cur2[1]] = false;
                minPq.poll();
                maxPq.poll();
                maxPq.offer(cur1);
                minPq.offer(cur2);
            }

            ans = Math.min(ans, res);
        }

        return ans;
    }
}
```

---

# 写在最后


![image.png](https://pic.leetcode.cn/1705770044-ZgBpez-image.png)

