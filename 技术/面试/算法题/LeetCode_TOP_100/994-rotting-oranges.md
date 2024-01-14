


[994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/description)

多源bfs的入门题

```c++ []
class Solution {

public:

    int dirs[4][2] = {
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    };

    int orangesRotting(vector<vector<int>>& grid) {
        int h = grid.size(), w = grid[0].size();
        vector<vector<int>> path(h, vector<int>(w, -1));

        // 多源bfs
        deque<pair<int, int>> deq;
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (grid[i][j] == 2) {
                    deq.push_back({i, j});
                    path[i][j] = 0;
                }
            }
        }

        int res = 0;
        while (!deq.empty()) {
            auto [y, x] = deq.front();
            deq.pop_front();
            for (int i = 0; i < 4; i++) {
                int ty = y + dirs[i][0];
                int tx = x + dirs[i][1];
                if (ty >= 0 && ty < h && tx >= 0 && tx < w) {
                    if (grid[ty][tx] == 1 && (path[ty][tx] == -1 || path[ty][tx] > path[y][x] + 1)) {
                        path[ty][tx] = path[y][x] + 1;
                        deq.push_back({ty, tx});
                    }
                }
            }
            res = max(res, path[y][x]);
        }

        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (grid[i][j] == 1 && path[i][j] == -1) {
                    return -1;
                }
            }
        }
        return res;
    }
};
```