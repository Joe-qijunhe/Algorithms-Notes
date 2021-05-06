[toc]

# 无向图

>   图是由一组顶点和一组能够将两个顶点相连的边组成的

>   在图中，路径是由边顺序连接的一系列顶点。简单路径是一条没有重复顶点的路径。环是一条至少含有一条且至少含有一条且起点和终点相同的路径。简单环是一条（除了起点和终点必须相同之外）不含重复顶点和边的环。路径或环的长度为其中所包含的边数

>   如果从任意一个顶点都存在一条路径到另一个任意顶点，这幅图是连通图。一幅非连通的图由若干连通的部分组成，它们都是其极大连通子图。

>   树是一幅无环连通图。互不相连的树组成的集合称为森林。连通图的生成树是它的一幅子图，它含有图中的所有顶点且是一棵树。图的生成树森林是它的所有连通子图的生成树的集合。

图处理代码

```java
    public static int degree(Graph G, int v) {
        int degree = 0;
        for (int w : G.adj(v))
            degree++;
        return degree;
    }

    public static int maxDegree(Graph G) {
        int max = 0;
        for (int v = 0; v < G.V(); v++) {
            if (degree(G, v) > max)
                max = degree(G, v);
        }
        return max;
    }

    public static double avgDegree(Graph G) {
        return 2.0 * G.E() / G.V();
    }

    public static int numberOfSelfLoops(Graph G) {
        int count = 0;
        for (int v = 0; v < G.V(); v++) {
            for (int w : G.adj(v)) {
                if (v == w)
                    count++;
            }
        }
        return count / 2;
    }
```

## Graph数据类型

性能特点：

-   使用的空间和V+E成正比
-   添加一条边的时间为常数
-   遍历顶点v的所有相邻顶点所需的时间和v的度数成正比

```java
import java.util.Scanner;

public class Graph {
    private final int V;
    private int E;
    private Bag<Integer>[] adj;

    public Graph(int V) {
        this.V = V;
        this.E = 0;
        adj = (Bag<Integer>[]) new Bag[V];
        for (int v = 0; v < adj.length; v++) {
            adj[v] = new Bag<Integer>();
        }
    }

    public Graph(Scanner in) {
        this(in.nextInt());
        int E = in.nextInt();
        for (int i = 0; i < E; i++) {
            int v = in.nextInt();
            int w = in.nextInt();
            addEdge(v, w);
        }

    }

    public int V() {
        return V;
    }

    public int E() {
        return E;
    }

    public void addEdge(int v, int w) {
        adj[v].add(w);
        adj[w].add(v);
        E++;
    }

    public Iterable<Integer> adj(int v) {
        return adj[v];
    }

    public String toString() {
        String s = V + " vertices, " + E + " edges\n";
        for (int v = 0; v < V; v++) {
            s += v + ": ";
            for (int w : adj(v)) {
                s += w + " ";
            }
            s += "\n";
        }
        return s;
    }
}

```

添加一个复制构造函数

```java
    public Graph(Graph G) {
        this.V = G.V();
        this.E = G.E();
        if (V < 0) throw new IllegalArgumentException("Number of vertices must be non-negative");

        // update adjacency lists
        adj = (Bag<Integer>[]) new Bag[V];
        for (int v = 0; v < V; v++) {
            adj[v] = new Bag<Integer>();
        }

        for (int v = 0; v < G.V(); v++) {
            // reverse so that adjacency list is in same order as original
            Stack<Integer> reverse = new Stack<Integer>();
            for (int w : G.adj[v]) {
                reverse.push(w);
            }
            for (int w : reverse) {
                adj[v].add(w);
            }
        }
    }
```

## 深度优先搜索

>   深度优先搜索标记与起点连通的所有顶点所需的时间和顶点的度数之和成正比

解决连通性的问题：回答了”两个给定的顶点是否连通“（marked方法）

算法遍历边和访问顶点的顺序与图的表示是有关的

```java
public class DepthFirstSearch {
    private boolean[] marked;
    private int count;

    public DepthFirstSearch(Graph G, int s) {
        marked = new boolean[G.V()];
        dfs(G,s);
    }

    private void dfs(Graph G, int v) {
        marked[v] = true;
        count++;
        for (int w : G.adj(v)) {
            if (!marked(w))
                dfs(G, w);
        }
    }

    public boolean marked(int w) {
        return marked[w];
    }

    public int count() {
        return count;
    }

    public static void main(String[] args) {
        Graph graph = new Graph(6);
        graph.addEdge(0,5);
        graph.addEdge(0,1);
        graph.addEdge(0,2);
        graph.addEdge(2,1);
        graph.addEdge(2,3);
        graph.addEdge(2,4);
        graph.addEdge(3,5);
        graph.addEdge(3,4);
        DepthFirstSearch dfs = new DepthFirstSearch(graph, 0);
        for (int v = 0; v < graph.V(); v++){
            if (dfs.marked(v)){
                System.out.print(v + " ");
            }
        }
        System.out.println();
        if (dfs.count() != graph.V()){
            System.out.println("Not connected.");
        }
    }
}

```

### 使用深度优先搜索查找图中的路径

解决单点路径问题：给定一幅图和一个起点s，回答”从s到给定目的的顶点v是否存在一条路径？如果有，找出这条路径。“

在之前的代码的基础上添加了一个edgeTo[]数组，这个数组可以找到从每个与s连通的顶点回到s的路径。在由边v-w第一次访问任意w时，将edge[w]设为v来记住这条路径。换句话说，v-w是从s到w的路径上的最后一条已知的边。

```java
import Chapter1.Stack;

public class DepthFirstPaths {
    private boolean[] marked;   //这个顶点上调用过dfs了吗？
    private int[] edgeTo;   //从起点到一个顶点的已知路径上的最后一个顶点
    private final int s;    //起点

    public DepthFirstPaths(Graph G, int s) {
        marked = new boolean[G.V()];
        edgeTo = new int[G.V()];
        this.s = s;
        dfs(G, s);
    }

    public void dfs(Graph G, int v) {
        marked[v] = true;
        for (int w : G.adj(v)) {
            if (!marked[w]) {
                edgeTo[w] = v;
                dfs(G, w);
            }
        }
    }

    public boolean hasPathTo(int v) {
        return marked[v];
    }

    public Iterable<Integer> pathTo(int v) {
        if (!hasPathTo(v)) return null;
        Stack<Integer> path = new Stack<>();
        for (int x = v; x != s; x = edgeTo[x])
            path.push(x);
        path.push(s);
        return path;
    }

    public static void main(String[] args) {
        Graph graph = new Graph(6);
        graph.addEdge(0, 5);
        graph.addEdge(0, 1);
        graph.addEdge(0, 2);
        graph.addEdge(2, 1);
        graph.addEdge(2, 3);
        graph.addEdge(2, 4);
        graph.addEdge(3, 5);
        graph.addEdge(3, 4);
        int s = 0;
        DepthFirstPaths dsp = new DepthFirstPaths(graph, s);
        for (int v = 0; v < graph.V(); v++) {
            System.out.print(s + " to " + v + " : ");
            if (dsp.hasPathTo(v)) {
                for (int x : dsp.pathTo(v)) {
                    if (x == s)
                        System.out.print(x);
                    else
                        System.out.print("-" + x);
                }
            }
            System.out.println();
        }
    }
}
```

### 连通分量

深度优先搜索的下一个直接应用时找出一幅图的所有连通分量，将图中所有的顶点切分为等价类（连通分量）。

递归的第一次调用的参数是顶点0---他会标记所有与0连通的顶点。在id[]中，标识符0会被赋予第一个连通分量的所有顶点，1会被赋予第二个连通分量的所有顶点，以此类推。接着，for循环会查找每个没有被标记的顶点并递归调用dfs()来标记和它相邻的所有顶点。

```java
import Chapter1.Bag;

public class CC {
    private boolean[] marked;
    private int[] id;
    private int count;

    public CC(Graph G) {
        marked = new boolean[G.V()];
        id = new int[G.V()];
        for (int s = 0; s < G.V(); s++) {
            if (!marked[s]) {
                dfs(G, s);
                count++;
            }
        }
    }

    private void dfs(Graph G, int v) {
        marked[v] = true;
        id[v] = count;
        for (int w : G.adj(v)) {
            if (!marked[w]) {
                dfs(G, w);
            }
        }
    }

    public boolean connected(int v, int w) {
        return id[v] == id[w];
    }

    public int id(int v) {
        return id[v];
    }

    public int count() {
        return count;
    }

    public static void main(String[] args) {
        Graph graph = new Graph(13);
        graph.addEdge(0, 1);
        graph.addEdge(0, 2);
        graph.addEdge(0, 6);
        graph.addEdge(0, 5);
        graph.addEdge(6, 4);
        graph.addEdge(3, 4);
        graph.addEdge(3, 5);
        graph.addEdge(4, 5);
        graph.addEdge(7, 8);
        graph.addEdge(9, 10);
        graph.addEdge(9, 11);
        graph.addEdge(9, 12);
        graph.addEdge(11, 12);
        CC cc = new CC(graph);
        int M = cc.count();
        System.out.println(M + " components");

        Bag<Integer>[] components = (Bag<Integer>[]) new Bag[M];
        for (int i = 0; i < M; i++)
            components[i] = new Bag<>();
        for (int v = 0; v < graph.V(); v++)
            components[cc.id(v)].add(v);
        for (int i = 0; i < M; i++) {
            for (int v : components[i]) {
                System.out.print(v + " ");
            }
            System.out.println();
        }
    }
}
```

### 无环图判断

dfs(Graph G, int u, int v) v代表当前结点，u代表上一个结点。进行深度优先搜索，走到一条线的最后一个结点时（意味着相邻的w都被mark过了（如果有没被mark过的点会继续dfs）），如果他相邻的结点只有上一个结点，那这条路线无环。但如果相邻的结点有其他的被mark过的点说明这条路线有环。

```java
public class Cycle {
    private boolean[] marked;
    private int[] edgeTo;
    private Stack<Integer> cycle;

    /**
     * Determines whether the undirected graph {@code G} has a cycle and,
     * if so, finds such a cycle.
     *
     * @param G the undirected graph
     */
    public Cycle(Graph G) {
        if (hasSelfLoop(G)) return;
        if (hasParallelEdges(G)) return;
        marked = new boolean[G.V()];
        edgeTo = new int[G.V()];
        for (int v = 0; v < G.V(); v++)
            if (!marked[v])
                dfs(G, -1, v);
    }


    // does this graph have a self loop?
    // side effect: initialize cycle to be self loop
    private boolean hasSelfLoop(Graph G) {
        for (int v = 0; v < G.V(); v++) {
            for (int w : G.adj(v)) {
                if (v == w) {
                    cycle = new Stack<Integer>();
                    cycle.push(v);
                    cycle.push(v);
                    return true;
                }
            }
        }
        return false;
    }

    // does this graph have two parallel edges?
    // side effect: initialize cycle to be two parallel edges
    private boolean hasParallelEdges(Graph G) {
        marked = new boolean[G.V()];

        for (int v = 0; v < G.V(); v++) {

            // check for parallel edges incident to v
            for (int w : G.adj(v)) {
                if (marked[w]) {
                    cycle = new Stack<Integer>();
                    cycle.push(v);
                    cycle.push(w);
                    cycle.push(v);
                    return true;
                }
                marked[w] = true;
            }

            // reset so marked[v] = false for all v
            for (int w : G.adj(v)) {
                marked[w] = false;
            }
        }
        return false;
    }

    /**
     * Returns true if the graph {@code G} has a cycle.
     *
     * @return {@code true} if the graph has a cycle; {@code false} otherwise
     */
    public boolean hasCycle() {
        return cycle != null;
    }

     /**
     * Returns a cycle in the graph {@code G}.
     * @return a cycle if the graph {@code G} has a cycle,
     *         and {@code null} otherwise
     */
    public Iterable<Integer> cycle() {
        return cycle;
    }

    private void dfs(Graph G, int u, int v) {
        marked[v] = true;
        for (int w : G.adj(v)) {

            // short circuit if cycle already found
            if (cycle != null) return;

            if (!marked[w]) {
                edgeTo[w] = v;
                dfs(G, v, w);
            }

            // check for cycle (but disregard reverse of edge leading to v)
            else if (w != u) {
                cycle = new Stack<Integer>();
                for (int x = v; x != w; x = edgeTo[x]) {
                    cycle.push(x);
                }
                cycle.push(w);
                cycle.push(v);
            }
        }
    }

    /**
     * Unit tests the {@code Cycle} data type.
     *
     * @param args the command-line arguments
     */
    public static void main(String[] args) {
        In in = new In(args[0]);
        Graph G = new Graph(in);
        Cycle finder = new Cycle(G);
        if (finder.hasCycle()) {
            for (int v : finder.cycle()) {
                StdOut.print(v + " ");
            }
            StdOut.println();
        }
        else {
            StdOut.println("Graph is acyclic");
        }
    }
}
```

### 二分图判断

颜色true的为一边，颜色false的为一边。深度优先搜索往下走时，把下一个点标记成与当前点不同的颜色（color[w] = !color[v];）。走到最后一个点时，如果他相邻的被marked过的点有颜色和他一样的（意味着他和同一边的点有路径），那么这就不是二分图

```java
package Graph;

public class TwoColor {
    private boolean[] marked;
    private boolean[] color;
    private boolean isTwoColorable = true;

    public TwoColor(Graph G) {
        marked = new boolean[G.V()];
        color = new boolean[G.V()];
        for (int s = 0; s < G.V(); s++) {
            if (!marked[s]) {
                dfs(G, s);
            }
        }
    }

    private void dfs(Graph G, int v) {
        marked[v] = true;
        for (int w : G.adj(v)){
            if (!marked[w]){
                color[w] = !color[v];
                dfs(G,w);
            }
            else if (color[w] == color[v]){
                isTwoColorable = false;
            }
        }
    }
    
    public boolean isBipartite(){
        return isTwoColorable;
    }
}

```

### 割点与桥（割边）

[原文](https://www.cnblogs.com/nullzx/p/7968110.html)

在无向图中才有割边和割点的定义

-   割点：无向连通图中，去掉一个顶点及和它相邻的所有边，图中的连通分量数增加，则该顶点称为割点。

-   桥（割边)：无向联通图中，去掉一条边，图中的连通分量数增加，则这条边，称为桥或者割边。

割点与桥（割边）的关系：

1）有割点不一定有桥,有桥一定存在割点

2）桥一定是割点依附的边。

下图中顶点C为割点，但和C相连的边都不是桥。

![bridge1](..\src\bridge\bridge1.png)



暴力解法

在图中去掉某个顶点，然后进行DFS遍历，如果连通分量增加，那么该顶点就是割点。如果在图中去掉某条边，然后进行DFS遍历，如果连通分量增加，那么该边就是割边。化的暴力解法是只测试在DFS路径上的边。

在具体的代码实现中，并不需要真正删除该顶点和删除依附于该顶点所有边。对于割点，我们只需要在DFS前，将该顶点对应是否已访问的标记置为ture，然后从其它顶点为根进行DFS即可。对于割边，我们只需要禁止从这条边进行DFS后，如果联通分量增加了，那么这条边就是割边。

Tarjan算法

假设DFS中我们从顶点U访问到了顶点V（此时顶点V还未被访问过），那么我们称顶点U为顶点V的父顶点，V为U的孩子顶点。在顶点U之前被访问过的顶点，我们就称之为U的祖先顶点。

显然，如果顶点U的**所有孩子顶点**可以不通过父顶点U，而访问到U的祖先顶点，那么说明此时去掉顶点U不影响图的连通性，U就不是割点。相反，如果顶点U**至少存在一个孩子顶点**，必须通过父顶点U才能访问到U的祖先顶点，那么去掉顶点U后，顶点U的祖先顶点和孩子顶点就不连通了，说明U是一个割点。

箭头表示DFS访问的顺序（而不表示有向图，对于顶点D而言，D的孩子顶点可以通过连通区域1红色的边回到D的祖先顶点C（此时C已被访问过），所以此时D不是割点：

![bridge2](..\src\bridge\bridge2.png)

连通区域2中的顶点，必须通过D才能访问到D的祖先顶点，所以说此时D为割点：

![bridge3](..\src\bridge\bridge3.png)

在具体实现Tarjan算法上，我们需要在DFS（深度优先遍历）中，额外定义三个数组

-   dnf数组的下标表示顶点的编号，数组中的值表示该顶点在DFS中的遍历顺序(或者说时间戳)，每访问到一个未访问过的顶点，访问顺序的值（时间戳）就增加1。**子顶点的dfn值一定比父顶点的dfn值大**（但不一定恰好大1，比如父顶点有两个及两个以上分支的情况）。在访问一个顶点后，它的dfn的值就确定下来了，不会再改变。

-   low数组的下标表示顶点的编号，数组中的值表示DFS中该顶点**不通过父顶点**能访问到的祖先顶点中最小的顺序值（或者说时间戳）。

    每个顶点初始的low值和dfn值应该一样，在DFS中，我们根据情况不断更新low的值。

    假设由顶点U访问到顶点V。当从顶点V回溯到顶点U时，如果 dfn[v] < low[u]，那么low[u] = dfn[v]

    如果顶点U还有它分支，每个分支回溯时都进行上述操作，那么顶点low[u]就表示了不通过顶点U的父节点所能访问到的最早祖先节点。

-   parent数组，下标表示顶点的编号，数组中的值表示该顶点的父顶点编号，它主要用于更新low值的时候排除父顶点，当然也可以其它的办法实现相同的功能。

例子

![bridge4](..\src\bridge\bridge4.png)

顶点I访问顶点D，而此时发现D已被访问，当从D回溯到I时，由于dfn[D] < low[I]。说明D是I的祖先顶点，所以到现在为止，顶点I不经过父顶点H能访问到的小时间戳为4。

![bridge5](..\src\bridge\bridge5.png)

从顶点I回到顶点H，显然到目前为止顶点H能访问到的最小时间戳也是4（因为我们到现在为止只知道能从H可以通过I访问到D）

![bridge6](..\src\bridge\bridge6.png)

![bridge7](..\src\bridge\bridge7.png)

![bridge8](..\src\bridge\bridge8.png)

由DFS访问顶点B，dfn[J] > dfn[B]，B为祖先顶点，顶点J不经过父顶点H能访问到的最早时间戳就是dfn[B]，即low[J] = 2

![bridge8](..\src\bridge\bridge8.png)

![bridge9](..\src\bridge\bridge9.png)

从顶点J回溯到顶点H，显然到目前为止顶点H能访问到的最早时间戳就更新为2（因为我们到现在为止知道了能从H访问到J），所以low[H] = 2

![bridge10](..\src\bridge\bridge10.png)

![bridge12](..\src\bridge\bridge12.png)

割点：判断顶点U是否为割点，用U顶点的dnf值和它的所有的孩子顶点的low值进行比较，如果存在至少一个孩子顶点V满足low[v] >= dnf[u]，就说明顶点V访问顶点U的祖先顶点，必须通过顶点U，而不存在顶点V到顶点U祖先顶点的其它路径，所以顶点U就是一个割点。对于没有孩子顶点的顶点，显然不会是割点。

桥（割边）：low[v] > dnf[u] 就说明V-U是桥

所以，顶点B,顶点E和顶点K为割点，A-B以及E-K和K-L为割边

```java
public class Bridge {
    private int bridges;      // number of bridges
    private int cnt;          // counter
    private int[] pre;        // pre[v] = order in which dfs examines v
    private int[] low;        // low[v] = lowest preorder of any vertex connected to v

    public Bridge(Graph G) {
        low = new int[G.V()];
        pre = new int[G.V()];
        for (int v = 0; v < G.V(); v++)
            low[v] = -1;
        for (int v = 0; v < G.V(); v++)
            pre[v] = -1;
        
        for (int v = 0; v < G.V(); v++)
            if (pre[v] == -1)
                dfs(G, v, v);
    }

    public int components() { return bridges + 1; }

    private void dfs(Graph G, int u, int v) {
        pre[v] = cnt++;
        low[v] = pre[v];
        for (int w : G.adj(v)) {
            if (pre[w] == -1) {
                dfs(G, v, w);
                low[v] = Math.min(low[v], low[w]);
                if (low[w] == pre[w]) {
                    StdOut.println(v + "-" + w + " is a bridge");
                    bridges++;
                }
            }

            // update low number - ignore reverse of edge leading to v
            else if (w != u)
                low[v] = Math.min(low[v], pre[w]);
        }
    }
}
```

## 广度优先搜索

解决单点最短路径：给定一幅图和一个起点s，回答“从s到给定目的的顶点v是否存在一条路径？如果有，找出最短的那条（含边数最少）”

深度优先搜素中，我们用了后进先出的栈。在广度优先搜素中，希望按照与起点的距离的顺序来遍历所有顶点，用先进先出的队列即可。

深度优先搜索探索一幅图的方式是寻找离起点更远的顶点，只有在碰到死胡同时才访问近处的顶点。广度优先搜索则会首先覆盖起点附近的顶点，只在临近的所有顶点都被访问了之后才向前进。

```java
import Chapter1.Queue;
import Chapter1.Stack;

public class BreadthFirstPaths {
    private boolean[] marked;
    private int[] edgeTo;
    private final int s;

    public BreadthFirstPaths(Graph G, int s) {
        marked = new boolean[G.V()];
        edgeTo = new int[G.V()];
        this.s = s;
        bfs(G, s);
    }

    private void bfs(Graph G, int s) {
        Queue<Integer> queue = new Queue<>();
        marked[s] = true;   //标记起点
        queue.enqueue(s);   //将它加入队列
        while (!queue.isEmpty()) {
            int v = queue.dequeue();    //从队列删去下一顶点
            for (int w : G.adj(v)) {
                if (!marked[w]) {   //对于每个未被标记的相邻顶点
                    edgeTo[w] = v;  //保存最短路径的最后一条边
                    marked[w] = true;   //标记
                    queue.enqueue(w);   //将它添加到队列
                }
            }
        }
    }

    public boolean hasPathTo(int v) {
        return marked[v];
    }

    public Iterable<Integer> pathTo(int v) {
        if (!hasPathTo(v)) return null;
        Stack<Integer> path = new Stack<>();
        for (int x = v; x != s; x = edgeTo[x])
            path.push(x);
        path.push(s);
        return path;
    }

    public static void main(String[] args) {
        Graph graph = new Graph(6);
        graph.addEdge(0, 5);
        graph.addEdge(0, 1);
        graph.addEdge(0, 2);
        graph.addEdge(2, 1);
        graph.addEdge(2, 3);
        graph.addEdge(2, 4);
        graph.addEdge(3, 5);
        graph.addEdge(3, 4);
        int s = 0;
        BreadthFirstPaths bfs = new BreadthFirstPaths(graph, s);
        for (int v = 0; v < graph.V(); v++) {
            System.out.print(s + " to " + v + " : ");
            if (bfs.hasPathTo(v)) {
                for (int x : bfs.pathTo(v)) {
                    if (x == s)
                        System.out.print(x);
                    else
                        System.out.print("-" + x);
                }
            }
            System.out.println();
        }
    }
}
```

### 间隔的度数

```java
public class DegreeOfSeparation {
    public static void main(String[] args) {
        SymbolGraph sg = new SymbolGraph("E:\\project\\IJ_Java\\Algorithms\\src\\Graph\\routes.txt", " ");
        Graph G = sg.G();
        String src = "JFK";
        if (!sg.contains(src)){
            System.out.println(src + " not in database.");
            return;
        }
        int s = sg.index(src);
        BreadthFirstPaths bfs = new BreadthFirstPaths(G, s);
        int t = sg.index("LAS");
        if (bfs.hasPathTo(t)){
            for (int v : bfs.pathTo(t)){
                System.out.println(" "+sg.name(v));
            }
        }else{
            System.out.println("Not connected");
        }
    }
}
```

## 符号图

在典型的应用中，图都是通过文件或网页定义的，使用的时字符串而非整数来表示和指代顶点。

符号表用到了一下三种数据结构

-   一个符号表st，键的类型为String（顶点名），值的类型为int（索引）
-   一个数组keys[]，用作反向索引，保存每个顶点索引所对应的顶点名
-   一个Graph对象G，它使用索引来引用图中顶点

SymbolGraph会遍历两边数据来构造以上结构，这主要是因为构造Graph对象需要顶点总数V。

```java
package Graph;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Scanner;

public class SymbolGraph {
    private HashMap<String, Integer> st;    //符号名 - 索引
    private String[] keys;  //索引 - 符号名
    private Graph G;

    public SymbolGraph(String stream, String sp) {
        st = new HashMap<>();
        Scanner in = null;
        try {
            in = new Scanner(new File(stream)); //第一遍
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        while (in.hasNextLine()) {
            String[] a = in.nextLine().split(sp);

            for (int i = 0; i < a.length; i++) {    //为每个不同的字符串关联一个索引
                if (!st.containsKey(a[i]))
                    st.put(a[i], st.size());
            }
        }
        keys = new String[st.size()];   //用来获得顶点名的反向索引是一个数组

        for (String name : st.keySet())
            keys[st.get(name)] = name;

        G = new Graph(st.size());

        try {
            in = new Scanner(new File(stream)); //第二遍
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        while (in.hasNextLine()) {
            String[] a = in.nextLine().split(sp);
            int v = st.get(a[0]);
            for (int i = 1; i < a.length; i++) {    //将每一行的第一个顶点和该行的其他顶点相连
                G.addEdge(v, st.get(a[i]));
            }
        }
    }

    public boolean contains(String s) {
        return st.containsKey(s);
    }

    public int index(String s) {
        return st.get(s);
    }

    public String name(int v) {
        return keys[v];
    }

    public Graph G() {
        return G;
    }

    public static void main(String[] args) {
        String fileName = "E:\\project\\IJ_Java\\Algorithms\\src\\Graph\\routes.txt";
        String delim = " ";
        SymbolGraph sg = new SymbolGraph(fileName, delim);
        Graph G = sg.G();
        String src = "JFK";
        for (int w : G.adj(sg.index(src))) {
            System.out.println(" " + sg.name(w));
        }
    }
}

```

