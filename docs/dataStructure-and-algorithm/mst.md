# 最小生成树

学习最小生成树算法之前我们先来了解下 下面这些概念：

**树**（Tree）：如果一个无向连通图中不存在回路，则这种图称为树。

**生成树** （Spanning Tree）：无向连通图G的一个子图如果是一颗包含G的所有顶点的树，则该子图称为G的生成树。

生成树是连通图的极小连通子图。这里所谓极小是指：若在树中任意增加一条边，则将出现一条回路；若去掉一条边，将会使之变成非连通图。

**最小生成树**（Minimum Spanning Tree，MST）：或者称为最小代价树Minimum-cost Spanning Tree, 对无向连通图的生成树，各边的权值总和称为生成树的权，权最小的生成树称为最小生成树。

构成生成树的准则有三条：

<1> 必须只使用该网络中的边来构造最小生成树。

<2> 必须使用且仅使用n-1条边来连接网络中的n个顶点

<3> 不能使用产生回路的边。 

构造最小生成树的算法主要有：克鲁斯卡尔（Kruskal）算法和普利姆（Prim）算法, 他们都遵循以上准则。

### Kruskal算法
算法描述：

1. 新建图G，G中拥有原图中相同的节点，但没有边；
2. 将原图中所有的边按权值从小到大排序；
3. 从权值最小的边开始，如果这条边连接的两个节点于图G中不在同一个连通分量中，则添加这条边到图G中；
4. 重复3，直至图G中所有的节点都在同一个连通分量中。

关于第3条：如何判断两个顶点是否在图G中处于一个联通分量？

在构建最小生成树的时候，需要一个数组存储每个顶点的终点，然后判断两个顶点的终点是否相同，如果不同，说明就不在同一个联通分量中

算法代码：

```java
import java.util.*;

public class Kruskal {
    class Edge {
        int start;
        int end;
        int weight;

        public Edge(int start, int end, int weight) {
            this.start = start;
            this.end = end;
            this.weight = weight;
        }

        @Override
        public String toString() {
            return "<" + start + "," + end + ">=" + weight;
        }
    }

    private static final int INF = Integer.MAX_VALUE;
    int[] data; // 顶点
    int[][] matrix; // 邻接矩阵

    //    List<Edge> edges; // 边
    public static void main(String[] args) {
        int[] data = new int[]{0, 1, 2, 3, 4, 5, 6};// 7个顶点
        // 二维数组表示邻接矩阵，用来描述顶点相连的边的权重
        int[][] matrix = new int[][]{
                {INF, 5, 7, INF, INF, INF, 2},
                {5, INF, INF, 9, INF, INF, 3},
                {7, INF, INF, INF, 8, INF, INF},
                {INF, 9, INF, INF, INF, 4, INF},
                {INF, INF, 8, INF, INF, 5, 4},
                {INF, INF, INF, 4, 5, INF, 6},
                {2, 3, INF, INF, 4, 6, INF}
        };
        Kruskal k = new Kruskal(data, matrix);
        k.kruskal();
    }

    public Kruskal(int[] data, int[][] matrix) {
        this.data = data;
        this.matrix = matrix;
    }

    public void kruskal() {
        List<Edge> edges = this.getEdges(matrix);
        int edgeNum = edges.size();
        sortEdges(edges);
        int[] ends = new int[edgeNum];
        // 创建结果数组，保存最后的最小生成树
        Edge[] res = new Edge[edgeNum];
        int count = 0;
        System.out.println(edges);
        // 遍历edges，判断边是否构成了回路，如果没有，就加入，否在就寻找下一个边
        for (int i = 0; i < edgeNum; i++) {
            Edge edge = edges.get(i);
            // 获取一条边的两个顶点的索引p1,p2
            int p1 = getIndex(edge.start);
            int p2 = getIndex(edge.end);
            // p1和p2的终点
            int end1 = getEnd(p1, ends);
            int end2 = getEnd(p2, ends);
            // 边的两个顶点的终点不同，加入不会构成回路，那么就可以加进去最小生成树
            if (end1 != end2) {
                ends[end1] = end2;
                res[count++] = edge;
            }
            // 已经加了顶点-1条边，那么就可以终止循环了
            if (count >= data.length - 1) break;
        }
        for (int i = 0; i < count; i++) {
            System.out.println(res[i]);
        }
    }

    // 获取顶点的索引
    private int getIndex(int v) {
        for (int i = 0; i < this.data.length; i++) {
            if (this.data[i] == v) {
                return i;
            }
        }
        return -1;
    }

    /**
     * 获取下标为i的顶点的终点
     *
     * @param i    下标为i的顶点
     * @param ends 记录了各个顶点的终点是哪个顶点（索引）
     * @return 下标为i的顶点的终点索引
     */
    private int getEnd(int i, int[] ends) {
        // 表示终点不是自身，因为初始化的时候ends[i]均为0
        // while循环是要一直找下去，直到找到终点
        while (ends[i] != 0) {
            i = ends[i];
        }
        return i;
    }

    // 将矩阵处理成边
    private List<Edge> getEdges(int[][] matrix) {
        List<Edge> res = new ArrayList<>();
        for (int i = 0; i < data.length; i++) {
            // 自己不连接自己 j=i+1
            for (int j = i + 1; j < data.length; j++) {
                // 不连通的顶点不生成边
                if (matrix[i][j] != INF) {
                    res.add(new Edge(data[i], data[j], matrix[i][j]));
                }
            }
        }
        return res;
    }

    // 对边排序,按照权重升序
    private void sortEdges(List<Edge> edges) {
        Collections.sort(edges, new Comparator<Edge>() {
            @Override
            public int compare(Edge a, Edge b) {
                return a.weight - b.weight;
            }
        });
    }
}
```

### Prim算法

算法描述

1).输入：一个加权连通图，其中顶点集合为V，边集合为E；

2).初始化：Vnew = {x}，其中x为集合V中的任一节点（起始点），Enew = {},为空；

3).重复下列操作，直到Vnew = V：

a.在集合E中选取权值最小的边<u, v>，其中u为集合Vnew中的元素，而v不在Vnew[集合](https://baike.baidu.com/item/集合/2908132)当中，并且v∈V（如果存在有多条满足前述条件即具有相同权值的边，则可任意选取其中之一）；

b.将v加入集合Vnew中，将<u, v>边加入集合Enew中；

4).输出：使用集合Vnew和Enew来描述所得到的[最小生成树](https://baike.baidu.com/item/最小生成树)

算法步骤

***Algorithm\***
**1)** 创建一个集合*mstSet* 用来跟踪MST中的所有顶点
**2)** 图的每个顶点的距离初始成正无穷. 第一个顶点距离初始为0以便第一个选中， dist={0,INF,INF,INF.....}
**3)** 当*mstSet*不包含图中所有的顶点时，重复下面的步骤
….**a)** 在不在*mstSet*集合的顶点中，选中一个顶点距离最小的顶点u
….**b)** 将u加入到 mstSet.
….**c)** 更新u相邻顶点的距离. 对于每一个相邻的顶点 *v*,如果 *u-v* 边的权重小于dist[v], dist[v]更新成weight(u, v)

算法图解

用下面的例子来说明算法步骤:
[![Fig-1](https://www.geeksforgeeks.org/wp-content/uploads/Fig-11.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/Fig-11.jpg)

*mstSet* 集合初始化为空，距离dist初始化为 {0, INF, INF, INF, INF, INF, INF, INF} . INF 表示无穷. 现在我们选出距离最小的顶点, 顶点0 被选中，将顶点0加入到*mstSet*.所以*mstSet*变成 {0}. 当0加入到*mstSet*之后，更新它相邻节点的距离. 相邻节点是1和7,1 和7的距离被更新为4和8. 下面的子图显示了顶点和他们的距离值, 仅仅显示有限距离的顶点， 在MST中的顶点用绿色标记

[![Fig-2](https://www.geeksforgeeks.org/wp-content/uploads/MST1.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/MST1.jpg)



选出不在MST中距离最小的顶点 ，顶点1被选中并且加入到*mstSet*. 所以*mstSet*现在变成了 {0, 1}. 更新顶点1的相邻顶点的距离. 顶点2的距离变成了8.

[![Fig-3](https://www.geeksforgeeks.org/wp-content/uploads/MST2.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/MST2.jpg)



选出不在MST中距离最小的顶点 ，顶点7被选中并且加入到*mstSet*. 所以 *mstSet*现在变成了{0, 1, 7}. 更新顶点7的相邻顶点距离. 顶点6和顶点 8 的距离分别变成了1和7.
[![Fig-4](https://www.geeksforgeeks.org/wp-content/uploads/MST3.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/MST3.jpg)

选出不在MST中距离最小的顶点 ，顶点6被选中. 所以  *mstSet*现在变成了{0, 1, 7 ,6}. 更新顶点6的相邻顶点的距离. 顶点5和顶点8放入距离被更新了



[![Fig-4](https://www.geeksforgeeks.org/wp-content/uploads/MST4.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/MST4.jpg)

重复上面的步骤直到 *mstSet* 包含图中所有的顶点. 最终，我们得到了下面的生成树



[![Fig-1](https://www.geeksforgeeks.org/wp-content/uploads/MST5.jpg)](https://www.geeksforgeeks.org/wp-content/uploads/MST5.jpg)



算法代码

<!-- tabs:start -->

### ****java****

```java
import java.util.Arrays;

// prim算法
public class Prim {
    private static final int INF = Integer.MAX_VALUE;
    private int V;
    private int[][] graph;
    public static void main(String[] args) {
        // 二维数组表示邻接矩阵，用来描述顶点相连的边的权重
        int[][] edges = new int[][]{
                {INF, 5, 7, INF, INF, INF, 2},
                {5, INF, INF, 9, INF, INF, 3},
                {7, INF, INF, INF, 8, INF, INF},
                {INF, 9, INF, INF, INF, 4, INF},
                {INF, INF, 8, INF, INF, 5, 4},
                {INF, INF, INF, 4, 5, INF, 6},
                {2, 3, INF, INF, 4, 6, INF}
        };
        Prim p = new Prim(edges);
        p.prim();
    }
    public Prim(int[][] graph) {
        V = graph.length;
        this.graph = graph;
    }
    public void prim() {
        int[] dist = new int[V];
        int parent[] = new int[V];
        Arrays.fill(parent, -1);
        boolean[] mstSet = new boolean[V];
        for (int i = 0; i < V; i++) {
            dist[i] = INF;
            mstSet[i] = false;
        }
        dist[0] = 0;//将第一个顶点距离初始化为0，所以会被第一个选中到MST中
        parent[0] = -1; // 第一个顶点是MST的根节点
        // MST有 V个顶点
        for (int i = 0; i < V - 1; i++) {
            // 从非MST的顶点中挑选出距离最小的点
            int u = minDistance(dist,mstSet);
            // 将顶点u加入MST
            mstSet[u] = true;
            // 更新u相邻顶点的dist
            for (int v = 0; v < V; v++) {
                // 更新dist[v]的三个必须条件:
                // v不在 MST中
                // v和u相邻
                // graph[u][v] < dist[v]
                if(!mstSet[v] && graph[u][v] > 0 && graph[u][v] < dist[v]) {
                    dist[v] = graph[u][v];
                    parent[v] = u;
                }
            }
        }
        printMST(parent);

    }
    private int minDistance(int[] dist, boolean[] mstSet) {
        int minDis = INF, minIndex = -1;
        for (int i = 0; i < V; i++) {
            if(!mstSet[i] && dist[i] < minDis) {
                minDis = dist[i];
                minIndex = i;
            }
        }
        return minIndex;
    }
    private void printMST(int[] parent)
    {
        System.out.println("Edge \tWeight");
        for (int i = 1; i < V; i++)
            System.out.println(parent[i] + " - " + i + "\t" + graph[i][parent[i]]);
    }


}

```

### **javascript**

```javascript
function Prim() {
    this.prim = function() {
        
    }
}
```

<!-- tabs:end -->
