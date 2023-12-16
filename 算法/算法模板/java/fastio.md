
# 快读板子

Java比较特殊，它的IO特别的复杂

最简单的Reader

```java
Scanner sc = new Scanner(System.in);
```

添加Buffered后的Reader

```java
Scanner sc = new Scanner(new BufferedInputStream(System.in));
```

但这样的还是不够

StringTokenizer方便对一行字符串进行切割

```java
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
```




