
###### DFS的基本流程：
需考虑如下几点：
<font color="green">起点（DFS入口）-- 扩散源点的某种规律 -- 确定信息的时机 -- 数据的结构 -- 扩散停止的时机</font>
1.(<font color="red">maxAreaOfIsland</font>)
给你一个大小为 `m x n` 的二进制矩阵 `grid` 。
**岛屿** 是由一些相邻的 `1` (代表土地) 构成的组合，这里的「相邻」要求两个 `1` 必须在 **水平或者竖直的四个方向上** 相邻。你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。
岛屿的面积是岛上值为 `1` 的单元格的数目。
计算并返回 `grid` 中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。
- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 50`
- `grid[i][j]` 为 `0` 或 `1`
```c++
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        //扩散用的方向数组
        int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
        //准备一个dfs函数
        auto dfs = [&](auto&& dfs, int x, int y)
        {
            //判断限制条件
            if(x < 0 || x == n || y < 0 || y == m || grid[x][y] == 0)
                return 0;
            //能过则要标记，到了新的一步
            grid[x][y] = 0;
            //这一步的有效信息，即面积加一
            int t = 1;
            //向四个方向扩散
            for(auto& [X,Y] : dirs)
                t += dfs(dfs,x+X,y+Y);
            return t;
        };
        //准备一个ans变量
        int ans = 0;
        //把所有岛屿都收集起来
        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
            {
                ans = max(ans, dfs(dfs,i,j));
            }
        //dfs,bfs搜索 注意极端情况
        //若全是岛屿或海洋，这个算法会不会出问题？
        //全是岛屿，也会dfs到，并放在ans中，全是海洋，则不会dfs，对应ans=0
        return ans; //故无需特判
    }
```
<font color="yellow">找到一个岛屿就搜一遍->dfs中有效信息->面积->搜的格子数->比较各个岛屿面积</font>


###### 有效信息（目的）的多样化：
1.
给定一个 `row x col` 的二维网格地图 `grid` ，其中：`grid[i][j] = 1` 表示陆地， `grid[i][j] = 0` 表示水域。
网格中的格子 **水平和垂直** 方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。
岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。
![[Pasted image 20241213203917.png]]
<font color="yellow">此处遍历整张图，有效信息为四个方向上是否有水</font>


2.
给你一个下标从 **0** 开始大小为 `m x n` 的二维整数数组 `grid` ，其中下标在 `(r, c)` 处的整数表示：
- 如果 `grid[r][c] = 0` ，那么它是一块 **陆地** 。
- 如果 `grid[r][c] > 0` ，那么它是一块 **水域** ，且包含 `grid[r][c]` 条鱼。
一位渔夫可以从任意 **水域** 格子 `(r, c)` 出发，然后执行以下操作任意次：
- 捕捞格子 `(r, c)` 处所有的鱼，或者
- 移动到相邻的 **水域** 格子。
请你返回渔夫最优策略下， **最多** 可以捕捞多少条鱼。如果没有水域格子，请你返回 `0` 。
格子 `(r, c)` **相邻** 的格子为 `(r, c + 1)` ，`(r, c - 1)` ，`(r + 1, c)` 和 `(r - 1, c)` ，前提是相邻格子在网格图内。
![[Pasted image 20241213204310.png]]
<font color="yellow">遍历整张图，遇到水域就dfs，有效信息为鱼->格点数字</font>


###### DFS的获取信息的时机
1.（<font color="red">colorBorder</font>）
给你一个大小为 `m x n` 的整数矩阵 `grid` ，表示一个网格。另给你三个整数 `row`、`col` 和 `color` 。网格中的每个值表示该位置处的网格块的颜色。
如果两个方块在任意 4 个方向上相邻，则称它们 **相邻** 。
如果两个方块具有相同的颜色且相邻，它们则属于同一个 **连通分量** 。
**连通分量的边界** 是指连通分量中满足下述条件之一的所有网格块：
- 在上、下、左、右任意一个方向上与不属于同一连通分量的网格块相邻
- 在网格的边界上（第一行/列或最后一行/列）
请你使用指定颜色 `color` 为所有包含网格块 `grid[row][col]` 的 **连通分量的边界** 进行着色。
并返回最终的网格 `grid`
<font color="yellow">此处由于返回grid，故grid也是有效信息，得新开vis查重数组，找到边界时改色(信息的时机)</font>
```c++
    vector<vector<int>> colorBorder(vector<vector<int>>& grid, int row, int col, int color) {
        int n = grid.size(), m = grid[0].size();
        //方向数组
        int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
        //查重数组
        bool vis[50][50];
        memset(vis,false,sizeof(vis));
        //存要改的位置
        vector<pair<int,int>> ans;
        auto dfs = [&](auto&& dfs, int x, int y) -> void
        {
            //判断到达的格子是否合理，这里颜色得相同，关系到两层dfs，故判断写后面
            //这样的话似乎判断写后面比较好
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                //判断
                //但这时候正是要染色的时机
                if(nx < 0 || nx == n || ny < 0 || ny == m || grid[nx][ny] != grid[x][y] || vis[nx][ny] == true)
                {
                    //因为原地改颜色会影响后面的数判断，把要改的取出来，后面再改（具体怎么影响我也不知道，调了半天）
                    if(nx < 0 || nx == n || ny < 0 || ny == m) //边界直接染色，要分开，不然vis越界
                        ans.emplace_back(x,y);
                    else if(vis[nx][ny] == false) //不是边界那就是不同或者重复，只要不同的
                        ans.emplace_back(x,y);
                    continue;
                }
                //扩散，记得标记
                vis[nx][ny] = true;
                dfs(dfs,nx,ny);
            } 
        };
        //初始化起点
        vis[row][col] = true;
        dfs(dfs,row,col);
        //改颜色
        for(auto [x,y] : ans)
            grid[x][y] = color;
        return grid;
    }
```


2.（<font color="red">containsCycle</font>）
给你一个二维字符网格数组 `grid` ，大小为 `m x n` ，你需要检查 `grid` 中是否存在 **相同值** 形成的环。
一个环是一条开始和结束于同一个格子的长度 **大于等于 4** 的路径。对于一个给定的格子，你可以移动到它上、下、左、右四个方向相邻的格子之一，可以移动的前提是这两个格子有 **相同的值** 。
同时，你也不能回到上一次移动时所在的格子。比方说，环  `(1, 1) -> (1, 2) -> (1, 1)` 是不合法的，因为从 `(1, 2)` 移动到 `(1, 1)` 回到了上一次移动时的格子。
如果 `grid` 中有相同值形成的环，请你返回 `true` ，否则返回 `false` 
- `m == grid.length`
- `n == grid[i].length`
- `1 <= m <= 500`
- `1 <= n <= 500`
- `grid` 只包含小写英文字母
<font color="yellow">（翻译）保证不往回走，找到走过的格子就是有环，故找的信息就是标记过的格点，且不回走</font>
<font color="yellow">保证不回走可以增加变量记录上一次的路，选择一个pair记录</font>
<font color="yellow">因为会有grid[][] != grid[][]的字母边界判断，不宜原数组标记，开vis</font>
```c++
    bool vis[500][500];
    int dirs[4][2] = {{1,0},{-1,0},{0,-1},{0,1}};
    bool containsCycle(vector<vector<char>>& grid)
    {
        int n = grid.size(), m = grid[0].size();
        memset(vis,false,sizeof(vis));

        bool ans = false;
        auto dfs = [&](auto&& dfs, int x, int y, pair<int,int> last) -> void
        {
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                //越界直接走，字母边界也直接走
                if(nx < 0 || nx == n || ny < 0 || ny == m || grid[nx][ny] != grid[x][y])
                    continue;
                //要是往回走了，直接走
                if(nx == last.first && ny == last.second)
                    continue;
                //要是没有越界，也不是新的字母，而是走到了之前的位置，就成功了
                if(vis[nx][ny])
                {
                    ans = true;
                    //此时无需继续下去，直接退出dfs
                    return;
                }
                //设置重复标记，以及pair
                vis[nx][ny] = true;
                dfs(dfs,nx,ny,pair<int,int>(x,y));
            }
        };

        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
            {
                if(!vis[i][j])
                {
                    vis[i][j] = true;
                    dfs(dfs,i,j,pair<int,int>(i,j));
                }  
            }

        return ans;
    }
```


###### DFS的缓存

给你一个大小为 `n x n` 二进制矩阵 `grid` 。**最多** 只能将一格 `0` 变成 `1` 。
返回执行此操作后，`grid` 中最大的岛屿面积是多少？
**岛屿** 由一组上、下、左、右四个方向相连的 `1` 形成
- `n == grid.length`
- `n == grid[i].length`
- `1 <= n <= 500`
- `grid[i][j]` 为 `0` 或 `1`
<font color="yellow">容易想到枚举每个零四个方向上岛的数量，但这会对一个岛重复DFS，故给每个岛编号（依据DFS次数），将有效信息存进去</font>
<font color="yellow">看代码，注释挺详细的</font>
```c++
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    //我们用DFS的次数来给岛屿编号，用area数组存储有效信息（岛面积），area的size表示完成的DFS次数
    //每DFS完一次，area会加入该次的答案，size+1，下次DFS时编号也对应+1

    //方便枚举0时找到四个方向上岛的编号，我们直接把原数组grid当vis存编号，因为gird里的信息不是目的
    //所以要保证编号大于1（size+2），不然grid会混乱，这里只有0,1数字，故可以用上面的操作；若是不能呢？
    //要从编号映射到对应岛屿的面积，用其编号-2就行
    vector<int> area;
    int largestIsland(vector<vector<int>>& grid)
    {
        int n = grid.size(), m = grid[0].size();

        auto dfs = [&](auto&& dfs, int x, int y) -> int
        {
            int res = 1;  //进了DFS表示已经走了一步，对应面积res += 1
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                //一定要写 != 1，不然会与编号混淆
                if(nx < 0 || nx == n || ny < 0 || ny == m || grid[nx][ny] != 1)
                    continue;
                //记得标记，这里要标记成编号,但不能与0,1重复
                grid[nx][ny] = area.size()+2;
                res += dfs(dfs,nx,ny);
            }
            return res;
        };

        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
            {
                if(grid[i][j] == 1)
                {
                    //入口的过程也是DFS的部分，因为已经判断合法将其进入，但要标记该步
                    grid[i][j] = area.size()+2;
                    area.push_back(dfs(dfs,i,j));
                }    
            }
        //编号完了，area也拿到了，开始枚举0四周的岛屿面积，找出最大值
        //由于0四周可能有相同的岛，还要考虑去重
        //我们用set存储添加了的岛的编号，若出现相同的就跳过
        unordered_set<int> cnt;
        //由于我们要变一个陆地来连接，故ans最后+1，别忘了
        int ans = 0;
        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
            {
                if(grid[i][j] == 0)
                {
                    //每次用完记得清空查询表
                    cnt.clear();
                    int sum = 0;
                    for(auto& [X,Y] : dirs)
                    {
                        int nx = i+X, ny = j+Y;
                        //忘了判断方向上是否越界了
                        if(nx < 0 || nx == n || ny < 0 || ny == m || grid[nx][ny] <= 1)
                            continue;
                        //编号入表，若插入失败则重复
                        if(cnt.insert(grid[nx][ny]).second == false)
                            continue;
                        sum += area[grid[nx][ny] - 2];
                    }
                    ans = max(ans, sum);
                }
            }
        //结束了吗？？？？
        //我们看看特殊情况
        //如果全是陆地：会DFS，但不会更新ans，矩阵面积就是答案（特殊情况）
        //如果全是水：我至少能变一个陆地（一般情况）
        //如果矩阵长度为1，被包括在上几条中
        //长度为0，没面积，不进入DFS或连接程序，因一般情况会+1，故要-1（但数据范围无此项，不管）
        return ans == 0 && grid[0][0] ? n*m : ans+1;
    }
```
