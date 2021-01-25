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

dfs(Graph G, int v, int u) v代表当前结点，u代表上一个结点。进行深度优先搜索，走到一条线的最后一个结点时（意味着相邻的w都被mark过了（如果有没被mark过的点会继续dfs）），如果他相邻的结点只有上一个结点，那这条路线无环。但如果相邻的结点有其他的被mark过的点说明这条路线有环。

```java
public class Cycle {
    private boolean[] marked;
    private boolean hasCycle;

    public Cycle(Graph G) {
        marked = new boolean[G.V()];
        for (int s = 0; s < G.V(); s++) {
            if (!marked[s]) {
                dfs(G, s, s);
            }
        }
    }

    private void dfs(Graph G, int v, int u) {
        marked[v] = true;
        for (int w : G.adj(v)) {
            if (!marked[w])
                dfs(G, w, v);
            else if (w != u)
                hasCycle = true;
        }
    }

    public boolean hasCycle(){
        return hasCycle;
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

