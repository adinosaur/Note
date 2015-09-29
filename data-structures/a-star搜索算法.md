a-star搜索算法
=========

###基本介绍
在玩游戏中，当我们点击了地图的一点后，系统通过寻路算法计算出NPC从当前方法（源点）到目标方格（终点）的最短路径，
然后便是移动根据求得的最短路径移动NPC。而a-star算法便是寻路算法中的一种。

###算法思想
a-star算法的核心思想比较好理解。假设地图被分为大小均等的网格，当我站在某一个位置x时，我的下一步可能是我附近的8个方格的其中一个。
当然这些方格可能有不能走的情况（比如游戏地图中经常有墙将地图分割开）。然后我要做的事情就是，判断身边的这8个格子，对每一个格子依据下面的两个标准作判断：

1. 从我现在的位置到达这个格子的代价。（G函数）
2. 这个格子离终点的估算距离。（H函数）

然后我综合这两个因素，可以简单地相加（F = G+H），得出一个综合两者的考虑，找出这之中最优的F便是我下一步应该走的位置。

###算法的文字描述
1. 把起点加入 open list 。
2. 重复如下过程：
    a. 遍历 open list ，查找 F 值最小的节点，把它作为当前要处理的节点。
    b. 把这个节点移到 close list 。
    c. 对当前方格的 8 个相邻方格的每一个方格？

        1 如果它是不可抵达的或者它在 close list 中，忽略它。否则，做如下操作。
        2 如果它不在 open list 中，把它加入 open list ，并且把当前方格设置为它的父亲，记录该方格的 F ， G 和 H 值。
        3 如果它已经在 open list 中，检查这条路径 ( 即经由当前方格到达它那里 ) 是否更好，用 G 值作参考。更小的 G 值表示这是更好的路径。如果是这样，把它的父亲设置为当前方格，并重新计算它的 G 和 F 值。如果你的 open list 是按F 值排序的话，改变后你可能需要重新排序。
    
    d. 停止，当你

        1 把终点加入到了 open list 中，此时路径已经找到了，或者
        2 查找终点失败，并且 open list 是空的，此时没有路径。

3. 保存路径。从终点开始，每个方格沿着父节点移动直至起点，这就是你的路径。


###代码实现
```
const int UNWALKABLE = 1;
const int SOURCE = 2;
const int DESTINATION = 3;

int dx[8] = {-1, -1, -1, 0, 0, 1, 1, 1};
int dy[8] = {-1, 0, 1, -1, 1, -1, 0, 1};
int w[8] = {14, 10, 14, 10, 10, 14, 10, 14};

// 地图上的每一个网格
class Point
{
    public:

        Point(int x, int y, int s=0):
            F(0),
            G(0),
            H(0),
            X(x),
            Y(y),
            status(s),
            parent(nullptr)
        {
        }

        int F;
        int G;
        int H;
    
        // 位置
        int X;
        int Y;

        // 点状态
        int status;

        // 父节点
        Point* parent;
};

class A_start
{
    public:

        A_start(int map[], int n, int m):
            _rows(n),
            _cols(m),
            _map()
        {
            init_map(map);
        }
        
        void init_map(int map[])
        {
            for (int i = 0; i != _rows; ++i)
            for (int j = 0; j != _cols; ++j)
            {
                _map.push_back(Point(i, j, map[i * _cols + j]));
            }
        }
        
        Point& position(int x, int y)
        {
            return _map.at(x * _cols + y);
        }
        
        // TODO 没对参数做检查
        bool inMap(int x, int y)
        {
            return x >= 0 && x < _rows &&
                    y >= 0 && y < _cols;
        }
        
        // TODO 没对参数做检查
        bool search(int sx, int sy, int ex, int ey)
        {
            position(sx, sy).status = SOURCE;
            position(ex, ey).status = DESTINATION;
            
            // 定义open_list和close_list
            list<Point*> open_list;
            unordered_set<Point*> close_list;
            
            // 初始化起点
            position(sx, sy).H = std::abs(ex - sx) * 10 + std::abs(ey - sy) * 10;
            position(sx, sy).F = position(sx, sy).H;
            open_list.push_back(&position(sx, sy));
            
            bool NotFound = true;
            
            while(NotFound && !open_list.empty())
            {   
                // TODO 笨方法 遍历open_list，从中取出F值最小的Point对象
                int min = (*open_list.begin())->F;
                list<Point*>::iterator min_it = open_list.begin();
                for (list<Point*>::iterator it = open_list.begin(); 
                    it != open_list.end(); ++it)
                    if (min > (*it)->F)
                    {
                        min = (*it)->F;
                        min_it = it;
                    }
                
                Point* u = *min_it;
                open_list.erase(min_it);
                // 将F值最小的添加进close_list
                close_list.insert(u);
                // 对当前方格周围的8个方格，通过计算F值，判断是否值得走
                for (int i = 0; i != 8; ++i)
                {
                    int pos_x = u->X + dx[i];
                    int pos_y = u->Y + dy[i];
                    
                    if (!inMap(pos_x, pos_y) || 
                        position(pos_x, pos_y).status == UNWALKABLE)
                        continue;
                    
                    Point* p = &position(pos_x, pos_y);
                    
                    // 到达目的地
                    if (pos_x == ex && pos_y == ey)
                    {
                        p->parent = u;
                        NotFound = false;
                        break;
                    }
                    
                    // 如果结点在close_list中忽略该结点
                    if (close_list.find(p) != close_list.end())
                        continue;
                    
                    else
                    {
                        // 如果它不在 open list 中，把它加入 open list ，并且把当前方格设置为它的父亲，记录该方格的 F,G,H 值。
                        if (find(open_list.begin(), open_list.end(), p) == open_list.end())
                        {
                            open_list.push_back(p);
                            
                            p->parent = u;
                            p->G = u->G + w[i];
                            p->H = std::abs(ex - pos_x) * 10 + std::abs(ey - pos_y) * 10;
                            p->F = p->G + p->H;
                        }
                        
                        // 如果它已经在 open_list 中，检查这条路径 ( 即经由当前方格到达它那里 ) 是否更好。
                        // 用 G 值作参考。更小的 G 值表示这是更好的路径。如果是这样，把它的父亲设置为当前方格，并重新计算它的 G 和 F 值。
                        // 如果你的 open_list 是按 F 值排序的话，改变后你可能需要重新排序。
                        else
                        {
                            if (p->G > u->G + w[i])
                            {
                                p->parent = u;
                                p->G = u->G + w[i];
                                p->F = p->G + p->H;
                            }
                        }
                    }
                } // for
            }
            
            // 打印对短路径
            Point* par = &position(ex, ey);
            while (par != nullptr)
            {
                cout << "(" << par->X << "," << par->Y << ")";
                if (par->parent != nullptr)
                    cout << " -> ";
                par = par->parent;
            }
            cout << endl;
            
            return !NotFound;
        }
        
        const int _rows;
        const int _cols;
        
    private:
        vector<Point> _map;
};

#endif

```

###参考资料
1. [A*寻路算法](http://www.cppblog.com/christanxw/archive/2006/04/07/5126.html)
