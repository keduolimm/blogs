


# 第 6 场 小白入门赛 解题报告 | 珂学家 | 简单场 + 元宵节日快乐

---
# 前言

---
# 整体评价

可以参考

---

## [A. 元宵节快乐](https://www.lanqiao.cn/problems/16990/learning/?contest_id=171)

题型: 签到

节日快乐，出题人也说出来自己的心愿, 祝大家AK快乐!

```java[]
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        System.out.println("Today AK!");
    }
}
```

---

## [B. 猜灯谜](https://www.lanqiao.cn/problems/16989/learning/?contest_id=171)

思路: 模拟

按题意模拟即可，环状结构

```java []
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            res.add(arr[(i - 1 + n) % n] + arr[(i + 1) % n]);
        }
        System.out.println(res.stream().map(String::valueOf).collect(Collectors.joining(" ")));
    }
    
}
```

---
## [C. 数学奇才](https://www.lanqiao.cn/problems/16991/learning/?contest_id=171)

思路: 思维题

可以贪心逆序从右到左翻转，每次翻转保证最右的非负数多一项，题意保证最多有n次。

所以，必然存在操作序列，使得数组元素全变非负形态。

$$\sum_{i=0}^{i=n-1} abs(a_i)$$

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr = new long[n];

        long sum = 0;
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
            sum += Math.abs(arr[i]);
        }
        System.out.println(sum);
    }

}
```

---
## [D. 你不干？有的是帕鲁干](https://www.lanqiao.cn/problems/16983/learning/?contest_id=171)

思路: 解方程

假设第一个元素为y, 那么

$$(y + 2) ^ 2 - y ^ 2 = 4(y + 1) = x$$

那么该y为

$$y=x/4 - 1$$

这边需要保证x是4的倍数，同时y必须为>=1的奇数

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int t = sc.nextInt();
        while (t-- > 0) {
            long x = sc.nextLong();
            if (x % 4 != 0) {
                System.out.println("No");
            } else {
                long y = x / 4 - 1;
                if (y <= 0 || y % 2 == 0) {
                    System.out.println("No");
                } else {
                    System.out.println("Yes");
                    System.out.println((y) + " " + (y + 2));
                }
            }
        }
    }

}
```

---
## [E. 等腰三角形](https://www.lanqiao.cn/problems/16981/learning/?contest_id=171)

思路: 贪心+双指针

感觉这题是这场周赛最难的，但是和历史比，算中等偏下的题

因为三角形中，边的关系需要满足, 

$$任意两边之和必须大于第三边$$

这题也是最大二分匹配模型

不过这题有个取巧的地方，就是

从底边出发

如果 $b_i < b_j$, 则选用$b_i$ 比 $b_j$ 的结果不会变差

因此可以对腰边和底边进行排序

然后使用双指针，进行贪心配对

```java []
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        int[] brr = new int[n];

        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            brr[i] = sc.nextInt();
        }
        Arrays.sort(arr);
        Arrays.sort(brr);
        // 双指针
        int res = 0;
        int j = 0;
        for (int i = 0; i < n; i++) {
            while (j < n && arr[j] * 2 <= brr[i]) {
                j++;
            }
            if (j < n) {
                j++;
                res++;
            }
        }
        System.out.println(res);
    }

}
```

---
## [F. 计算方程]()

---

# 写在最后
