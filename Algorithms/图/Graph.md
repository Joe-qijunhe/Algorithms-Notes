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

## 广度优先搜索

解决单点最短路径：给定一幅图和一个起点s，回答“从s到给定目的的顶点v是否存在一条路径？如果有，找出最短的那条（含边数最少）”

深度优先搜素中，我们用了后进先出的栈。在广度优先搜素中，希望按照与起点的距离的顺序来遍历所有顶点，用先进先出的队列即可。

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

