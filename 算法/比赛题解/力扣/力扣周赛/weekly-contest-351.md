# 第 351 场周赛 解题报告 | 珂学家 | 移项变形 + 状态机DP + 队列模拟


---

# 前言

![image.png](https://pic.leetcode.cn/1687665264-gCXKYr-image.png)

人们都害怕生离死别，怕的是痛苦与遗憾。

---

# 整体评价

唯一印象深刻, 就是T2, 还是很惧怕这类题型, 需要灵光一闪. T3是入门的状态机DP, T4很典, 力扣还是有很多这样的同类型题.

这端午节过的，老是各种花式看错题，比胡桃姐姐还惨.

---

## [T1. 美丽下标对的数目](https://leetcode.cn/problems/number-of-beautiful-pairs/)

因为数据范围小，所以全枚举左侧/右侧端点即可。

唯一要注意的是，不是数本身，而是第一个数字和最后一个数字。

```java
class Solution {
    
    int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }
    
    public int countBeautifulPairs(int[] nums) {
        int ans = 0;
        for (int i = 0; i < nums.length; i++) {
            int first = String.valueOf(nums[i]).charAt(0) - '0';
            for (int j = i + 1; j < nums.length; j++) {
                int second = String.valueOf(nums[j]).charAt(String.valueOf(nums[j]).length() - 1) - '0';
                if (gcd(first, second) == 1) {
                    ans++;
                }
            }
        }
        return ans;
    }
    
}
```

---

## [T2. 得到整数零需要执行的最少操作数](https://leetcode.cn/contest/weekly-contest-351/problems/minimum-operations-to-make-the-integer-zero/)

这题可以根据题意来变形

即 $ums1 - k * (2^i + nums2) = 0$

移项变形为

${nums1 - k * num2 = \sum 2^i}$

这边有个天然约束，就是右侧总共有k个$2^i$形态

令$s=nums1 - k * num2$

则满足${s>=k\ \&\&\ bitcount(s) <= k}$时，一定有解

无解的情况下，则是${s<=0\ \&\&\ num2 > 0}$, 可以直接剪枝退出。

所以这题的思路，移项变形，然后枚举K值。

至于为啥不TLE，因为num1可以取很大，而num2可以取1，-1

只能猜测，收敛很快，因为k值越大，其能构建数覆盖范围越广。

```java
class Solution {

    public int makeTheIntegerZero(int num1, int num2) {

        int k = 1;
        while (true) {
            long r = num1 - (long) num2 * k;

            if (num2 >= 0 && r < 0) {
                return -1;
            }

            if (r >= k && Long.bitCount(r) <= k) {
                return k;
            }
            k++;
        }

    }

}

```

---

## [T3. 将数组划分成若干好子数组的方式](https://leetcode.cn/contest/weekly-contest-351/problems/ways-to-split-array-into-good-subarrays/)

### 方法一：状态机DP



很经典的状态机DP

令 ${opt[i][s], i为前i个元素， s:0,1代表末尾的子串是否有1个1了}$

转移也非常的方便

1. 如果${nums[i] == 0}$

> ${opt[i][0] = opt[i - 1][0] + opt[i - 1][1]}$

- 和前一个没有1的子串合并 opt[i - 1][0]，
- 前一个子串已经有一个1了，即opt[i - 1][1]，则单独一个nums[i]=0单独成串

> ${opt[i][1] = opt[i - 1][1]}$

- nums[i]==0, 只能合并到 opt[i - 1][1] 中，形成opt[i][1]

---

2. 如果 ${nums[i] == 1}$

> ${opt[i][0] = 0}$

- nums[1]=1，则没法构建opt[i][0]状态，直接取0

> ${opt[i][1] = opt[i - 1][1] + opt[i - 1][0]}$

- 单独构建仅有nums[i]元素的子串于opt[i - 1][1]上
- 把nums[i]=1,合并到opt[i - 1][0]末尾的子串上，形成opt[i][1]

最终的结果为$opt[n - 1][1]$

```java
class Solution {
    
    public int numberOfGoodSubarraySplits(int[] nums) {

        long mod = 10_0000_0007l;
        int n = nums.length;

        long[][] opt = new long[n][2];
        
        opt[0][0] = (nums[0] == 0) ? 1 : 0;
        opt[0][1] = (nums[0] == 1) ? 1 : 0;
        
        for (int i = 1; i < n; i++) {
        
            int v = nums[i];
            
            if (v == 0) {
                opt[i][0] = (opt[i - 1][0] + opt[i - 1][1]) % mod;
                opt[i][1] = opt[i - 1][1];
            } else {
                opt[i][0] = 0;
                opt[i][1] = (opt[i - 1][0] + opt[i - 1][1]) % mod;
            }
            
        }
        
        return (int)(opt[n - 1][1] % mod);
        
    }
    
}
```

### 方法二: 插板法

就是在两个1之间，寻找断点的可能性$s_i$。最终的结果为 $\prod s_i$

而这个断点，你可以抽象为插板, $断点数量=两个1之间的0的个数 + 1$

```java
class Solution {
    
    public int numberOfGoodSubarraySplits(int[] nums) {
        // 特判case
        if (Arrays.stream(nums).sum() == 0) return 0;
        
        long mod = 10_0000_0007l;
        long ans = 1l;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 0) continue;
            // 找到第一个1
            for (int j = i + 1; j < nums.length; j++) {
                // 找到后续第一个1
                if (nums[j] == 1) {
                    ans = ans * (j - i) % mod;
                    break;
                }
            }
        }
        return (int)ans;
    }
    
}
```

---

## [T4. 机器人碰撞](https://leetcode.cn/contest/weekly-contest-351/problems/robot-collisions/)

很典的题，因为力扣上很多同类的题。

如果左侧的方向的机器人都是向左，那肯定和他们不会有交集了。

如果当前机器人方向是右，那肯定和左侧的机器人，不会有交集。

基于上述两者的观察，可以模拟实现。

用一个栈来维护左侧的向右行使机器人列表。

然后按位置顺序遍历进行比较即可。

```java
class Solution {

    // *) 单调栈
    public List<Integer> survivedRobotsHealths(int[] positions, int[] healths, String directions) {

        int n = positions.length;
        Integer[] arr = IntStream.range(0, n).boxed().toArray(Integer[]::new);
        Arrays.sort(arr, Comparator.comparing(t -> positions[t]));


        // *)
        List<int[]> result = new ArrayList<>();

        // *)
        Deque<int[]> deq = new LinkedList<>();

        for (int i = 0; i < n; i++) {
            int idx = arr[i];

            int p = positions[idx];
            int h = healths[idx];
            char dir = directions.charAt(idx);

            if (dir == 'L') {
                boolean f = false;
                while (!deq.isEmpty()) {
                    int[] cur = deq.pollLast();
                    if (cur[1] == h) {
                        // over
                        f = true;
                        break;
                    } else if (cur[1] > h) {
                        deq.offerLast(new int[] {cur[0], cur[1] - 1});
                        break;
                    } else {
                        h--;
                    }
                }
                if (!f && deq.isEmpty()) {
                    result.add(new int[] {idx, h});
                }
            } else {
                deq.offerLast(new int[] {idx, h});
            }
        }

        while (!deq.isEmpty()) {
            result.add(deq.poll());
        }

        Collections.sort(result, Comparator.comparing(x -> x[0]));

        return result.stream().map(x -> x[1]).collect(Collectors.toList());

    }

}
```


---

# 写在最后

力量若是到达了极限，接着考验的便是人心。

![image.png](https://pic.leetcode.cn/1687665203-xVIKwY-image.png)
