

[295. 数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/description)

这个对顶堆，写起来还挺麻烦的

```python []
class MedianFinder:

    def __init__(self):
        self.arr1 = []
        self.arr2 = []

    def addNum(self, num: int) -> None:
        if len(self.arr1) == len(self.arr2):
            heapq.heappush(self.arr1, -num)
        else:
            heapq.heappush(self.arr2, num)

        while len(self.arr1) > 0 and len(self.arr2) > 0 and -self.arr1[0] > self.arr2[0]:
            t1 = -heapq.heappop(self.arr1)
            t2 = heapq.heappop(self.arr2)
            heapq.heappush(self.arr1, -t2)
            heapq.heappush(self.arr2, t1)


    def findMedian(self) -> float:
        if len(self.arr1) > len(self.arr2):
            return -self.arr1[0]
        return (-self.arr1[0] + self.arr2[0]) / 2.0



# Your MedianFinder object will be instantiated and called as such:
# obj = MedianFinder()
# obj.addNum(num)
# param_2 = obj.findMedian()
```