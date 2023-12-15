# 牛客小白月赛76 解题报告 | 思维贪心场


---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230715/446702330_1689389435557/D2B5CA33BD970F64A6301FA75AE2EB22)

劳逸结合是不错，但也别放松过头。

---

# 题解

题面看起来是蛮简单的，但是做起来挺难的，前几题是思维题，就是易错。

对这场的C题，“怨念”很深，用枚举因子的思路来求解，但是一开始用了$O(\sqrt n)$的方法，结果TLE了，太惨了。

---
## [A. 猜拳游戏](https://ac.nowcoder.com/acm/contest/60393/A)

这题挺有意思的，竟然是$\color{red}“镜像”$

其实这个题意，以前小时候在玩的时候，好像也蹦出来类似的想法，应该是源于生活的题。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();
        System.out.println(s);
    }

}
```

---

## [B. Kevin喜欢一](https://ac.nowcoder.com/acm/contest/60393/B)

脑筋急转弯的题，贪心思路即可

首先就是翻倍扩展'1', 幂次方

只要幂次方大于等于n，就是解

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class B {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int cnt = 0;
            while ((1 << cnt) < n) {
                cnt++;
            }

            System.out.println(cnt);
        }
    }

}

```

---
## [C. A加B,A模B](https://ac.nowcoder.com/acm/contest/60393/C)

### 方法一：构建解

${a+b=n}$

${a\ \%\ b=m}$

因为a,b皆为非负数，因此可以推导得到

$a\ge m, b\gt m$

进而推导$(m+1)+m\le n$

也就是说 $2m>=n$ 时无解

若$2m\lt n$成立, 则令 $a=m, b=n-m$ 即为可行解

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        // 不使用快读的话，O(1)也在超时的边缘
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), m = sc.nextInt();
            if (2 * m >= n) {
                System.out.println(-1);
            } else {
                int a = m;
                int b = n - m;
                System.out.println(a + " " + b);
            }
        }
    }
    
}
```


### 方法二：正向构造因子解

正向解法，通过构造 数的因子，来求解

假设 $v=\prod p_i^{a_i}$

$p_i$为质数，$a_i$为指数

那正向构造数的因子的时间复杂度为 $\prod (a_i + 1)$

而$10^9$范围内，这个构造操作时间复杂度为$O(10^3)$

那拆解质因子呢？

对于$10^9$范围内，其实只要生成32000以内的质数表就行，这个可以预处理

而32000以内的质数个数，大概是3432个

所以拆解+正向构造枚举因子的整体时间复杂度为 $O(10^3)$

整体的流程为
- 预处理生成质数表
- 质因子拆解，生成(质因子，数）列表
- 正向构造因子

```c++
#include <cstdio>
#include <vector>
#include <cstring>
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

vector<int> primes;

const int sz = 32000;
bool vis[sz + 1];

void init() {

	memset(vis, false, sizeof(vis));
	for (int i = 2; i <= sz; i++) {
		if (vis[i]) continue;
		primes.push_back(i);
		for (long long j = (long)i * i; j <= sz; j += i) {
			vis[j] = true;
		}
			
	}
	
}

void split(int v, vector<pair<int,int> >& result) {

    int up = (int)sqrt(v);
    
	int len = primes.size();
	for (int i = 0; v > 1 && i < len; i++) {
		int u = primes[i];
        if (u > up) break;
 		if (v % u != 0) continue;
		int cnt = 0;
		while (v % u == 0) {
			v /= u;
			cnt++;
		}
		result.push_back(std::make_pair(u, cnt));
        
        up = (int)sqrt(v);
	}
	if (v > 1) {
		result.push_back(std::make_pair(v, 1));
	}

}

bool dfs(vector<pair<int, int> > factors, int s, int now, int d, int n, int m, int*a, int* b) {

	int other = d / now;
	if ((n - other) % other == m) {
		*a = n - other;
		*b = other;
		return true;
	}
	if (other <= m) return false;

	if (s >= factors.size()) return false;

	int f = factors[s].first, cnt = factors[s].second;

	int by = 1;
	for (int i = 0; i <= cnt; i++) {
		if (dfs(factors, s + 1, now * by, d, n, m, a, b)) {
			return true;
		} 
		by *= f;
	}

	return false;
}


int main(){

    // 预处理生成质因子表（只要生成32000以内就行了）
	init();

	int kase;
	scanf("%d", &kase);
	while (kase-- > 0) {
		int n, m;
		scanf("%d%d", &n, &m);
		if (m >= n) {
			printf("-1\n");
		} else {
			int d = n - m;
            
            // 对 n-m 进行质因子拆解 (factor, cnt)
			vector<pair<int, int> > factors;
			split(d, factors);

            // 反向构建n-m的因子，然后进行判定
			int a, b;
			if (dfs(factors, 0, 1, d, n, m, &a, &b)) {
				printf("%d %d\n", a, b);
			} else {
				printf("-1\n");
			}
		}	
	}
	return 0;
}
```
---

## [D. MoonLight的运算问题](https://ac.nowcoder.com/acm/contest/60393/D)

贪心即可，但是边界需要特殊处理下

就是当
$x=0,1，arr[i]=0,1$的时候，加比乘好

但是因为数很大，中间过程要取模，为了避免取模不小心生成 0,1，所以需要额外引入一个状态

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int kase =  sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
            }

            long mod = 998244353l;
            boolean f = false;
            long x = 0;

            // 贪心+讨论

            for (int i = 0; i < n; i++) {
                if (!f) {
                    if (x == 0 || x == 1 || arr[i] == 0 || arr[i] == 1) {
                        x += arr[i];
                    } else {
                        x *= arr[i];
                    }

                    // 这边x>=2, 就要状态迁移
                    if (x >= 2) {
                        f = true;
                        x %= mod;
                    }
                } else {
                    if (arr[i] == 0) {}
                    else if (arr[i] == 1) {
                        x = (x + 1) % mod;
                    } else {
                        x = x * arr[i] % mod;
                    }
                }
            }

            System.out.println(x);
        }
    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}

```

---

## [E. 括号序列操作专家](https://ac.nowcoder.com/acm/contest/60393/E)

括号序列问题，往往使用栈来模拟求解，这边其实也不例外。

对于合法性问题，相对比较简单，只要判断左右括号的数量是否相等即可

那交换次数如何求解呢？

其实可以继续用栈模拟的思路

如果栈中有'('多，则')'进行匹配，然后消掉最远的'('

如果栈中')'多，则'('需要找到未匹配的最远')', 然后匹配消掉

初看，感觉还是有些烦，需要借助一些数据结构进行维护。

实际上栈中，
- 要么都是'('
- 要么都是')'

而且都是匹配消掉的都是左端的，因此这个交互距离，就是 $delta=数量差 - 1$


```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            char[] str = sc.next().toCharArray();

            int left = 0, right = 0;
            for (int i = 0; i < str.length; i++) {
                if (str[i] == '(') {
                    left++;
                } else {
                    right++;
                }
            }

            if (left != right) {
                System.out.println("-1");
            } else {
                long ans = 0;
                // 不需要使用栈模拟，维护左右括号的计数即可
                int lb = 0, rb = 0;
                for (int i = 0; i < n; i++) {
                    if (str[i] == '(') {
                        if (rb > 0) {
                            ans += rb;
                            rb--;
                        } else {
                            lb++;
                        }
                    } else {
                        if (lb > 0) {
                            lb--;
                        } else {
                            rb++;
                        }
                    }
                }
                System.out.println(ans);
            }

        }

    }

}
```

---

## [F. Kevin的矩阵](https://ac.nowcoder.com/acm/contest/60393/F)

感觉这个矩阵有个贪心的上界，就是$\sqrt n$宽，$\sqrt n$高的类正方形矩阵形态

这样理论值为 $2*\sqrt n$

分类讨论下，
- 如果当前$m\gt \sqrt n$, 实际上改变某列值就行，贪心上界$\sqrt n$
- 如果当前$m\le \sqrt n$, 则往$\sqrt n$靠拢，因为存在贪心上界$2 * \sqrt n$

这样对于m列而言，如果从变化列数的角度出发，其实最多 $2*\sqrt n$, 因此这边的操作是枚举列数，然后统计每个列数下的操作数，取最小即可。

相当于把最优解，限定在某个范围内了。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), m = sc.nextInt(), k = sc.nextInt();

            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
            }

            int ans = Integer.MAX_VALUE;
            // 枚举长度，贪心

            int u = (int)Math.sqrt(n) + 1;

            for (int i = Math.max(1, m - 2 * u); i <= m + 2 * u; i++) {

                int mz = Integer.MAX_VALUE;
                int[] hash = new int[Math.min(n, i)];
                for (int j = 0; j < n; j++) {
                    hash[j % i] += (arr[j] != k ? 1 : 0);
                }
                for (int j = 0; j < hash.length; j++) {
                    mz = Math.min(mz, hash[j]);
                }

                ans = Math.min(ans, mz + Math.abs(i - m));
            }

            System.out.println(ans);
        }
    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}
```
---

## [G. Kevin逛超市](https://ac.nowcoder.com/acm/contest/60393/G)



---

# 写在最后

我没有午休的习惯，不过如果你累了，就去休息吧，三分钟之后我会叫醒你的。

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230715/446702330_1689389469553/D2B5CA33BD970F64A6301FA75AE2EB22)

