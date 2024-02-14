

```c++ []
struct random_hashmap {
    static uint64_t splitmix64(uint64_t x) {
        x ^= x << 13;
        x ^= x >> 7;
        x ^= x << 17;
        return x;
    }
    size_t operator () (uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count(); // 时间戳
        return splitmix64(x + FIXED_RANDOM);
    }
};
```