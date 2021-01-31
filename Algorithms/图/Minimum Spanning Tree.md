[toc]

# 最小生成树

加权图就是一种为每条边关联一个权值或是成本的图模型。

>   图的生成树是它的一棵含有其所有顶点的无环连通子图。一幅加权图的最小生成树是他的一棵权值（书中所有边的权值之和）最小的生成树。
>
>   树（无环连通图）最重要的性质
>
>   -   用一条边连接树中的任意两个顶点都会产生一个新的环
>   -   从树中删去一条边将会得到两棵独立的树

>   定义。图的一种切分是将图的所有顶点分为两个非空而且不重叠的两个集合。横切边是一条连接两个属于不同集合的顶点的边。

切分定理：

>   在一幅加权图中，给定任意的切分，它的横切边中的权重最小者必然属于图的最小生成树。

## 加权无向图

```java
public class Edge implements Comparable<Edge> {
    private final int v;    //顶点之一
    private final int w;    //另一个顶点
    private final double weight;    //边的权重

    public Edge(int v, int w, double weight) {
        this.v = v;
        this.w = w;
        this.weight = weight;
    }

    public double weight() {
        return weight;
    }

    public int either() {
        return v;
    }

    public int other(int vertex) {
        if (vertex == v)
            return w;
        else if (vertex == w)
            return v;
        return -1;
    }

    @Override
    public int compareTo(Edge that) {
        if (this.weight() < that.weight())
            return -1;
        else if (this.weight() > that.weight())
            return 1;
        else
            return 0;
    }

    public String toString() {
        return String.format("%d-%d %.2f", v, w, weight);
    }
}
```

```java
import Chapter1.Bag;

public class EdgeWeightedGraph {
    private final int V;    //顶点总数
    private int E;  //边的总数
    private Bag<Edge>[] adj;    //邻接表

    public EdgeWeightedGraph(int V) {
        this.V = V;
        this.E = 0;
        adj = (Bag<Edge>[]) new Bag[V];
        for (int v = 0; v < V; v++) {
            adj[v] = new Bag<>();
        }
    }

    public int V(){
        return V;
    }

    public int E(){
        return E;
    }

    public void addEdge(Edge e){
        int v = e.either(), w = e.other(v);
        adj[v].add(e);
        adj[w].add(e);
        E++;
    }

    public Iterable<Edge> adj(int v){
        return adj[v];
    }

    public Iterable<Edge> edges(){
        Bag<Edge> b = new Bag<>();
        for (int v = 0; v < V; v++){
            for (Edge e : adj[v]){
                //防止重复，如果另一顶点比自己小说明前面加入过了
                if (e.other(v) > v) b.add(e);
            }
        }
        return b;
    }
}

```



## Primer算法

### 延时实现

visit()来为树添加一个顶点，并将它标记为“已访问”并将与它关联的所有未失效的边加入优先队列，以保证队列含有所有connect树顶点with非树顶点的边（也可能含有一些已经失效的边）

内循环中，从优先队列中取出一条边并将它添加到树中（如果它未失效），再把这条边的另一个顶点也添加到树中，然后用新顶点作为参数调用visit()方法来更新横切边的集合。

```java
import Chapter1.Queue;
import Sort.MinPQ;

public class LazyPrimMST {
    private double weight;  //total weight of MST
    private boolean[] marked;   //最小生成树的顶点
    private Queue<Edge> mst;    //最小生成树的边
    private MinPQ<Edge> pq; //横切边

    public LazyPrimMST(EdgeWeightedGraph G){
        marked = new boolean[G.V()];
        mst = new Queue<>();
        pq = new MinPQ<>();
        visit(G, 0);    //假设G是连通的
        while (!pq.isEmpty()){
            Edge e = pq.delMin();   //从pq中得到权重最小的边
            int v = e.either(), w = e.other(v);
            if (marked[v] && marked[w]) continue;   //跳过失效的边
            mst.enqueue(e);
            weight += e.weight();
            if (!marked[v]) visit(G, v);    //将顶点（v或w）添加到树种
            if (!marked[w]) visit(G, w);
        }
    }

    private void visit(EdgeWeightedGraph G, int v){
        //标记顶点v并将所有连接v和未被标记顶点的边加入pq
        marked[v] = true;
        for (Edge e : G.adj(v)){
            if (!marked[e.other(v)])
                pq.insert(e);
        }
    }
    
    public double weight(){
        return weight;
    }

    public Iterable<Edge> edges(){
        return mst;
    }
}

```

### 及时实现

我们感兴趣的只是connect树顶点with非树顶点中权重最小的边。不需要在优先队列保存所有从w到树顶点的边，只需要保存权重最小的那条（与树中某个顶点连接起来的权重最小的那条），生成树扩大时再检查需不需要更新这条权重最小的边。

PrimeST会从优先队列取出一个顶点v并检查它的邻接表中的每条边v-w。如果w已经被标记过，这条边失效；如果w不在优先队列中或者v-w的权重小于目前已知的最小值edgeTo[w]，更新数组，将v-w作为将w和树连接的最佳选择。

索引优先队列：Key[]中index代表顶点 value代表该顶点到树的最小权重 ，pq根据权重进行排序，delMin()会（int min = pq[1]; keys[min] = null; return min; ）返回Key[]中的index（即返回下一个加入树的点）

```java
import Chapter1.Queue;
import Sort.IndexMinPQ;

public class PrimeMST {
    private Edge[] edgeTo;  //距离树最近的边
    private double[] distTo;    //distTo[w] = edgeTo[w].weight
    private boolean[] marked;   //如果v在树中则为true
    private IndexMinPQ<Double> pq;  //有效的横切边

    public PrimeMST(EdgeWeightedGraph G){
        edgeTo = new Edge[G.V()];
        distTo = new double[G.V()];
        marked = new boolean[G.V()];
        for (int v = 0; v < G.V(); v++){
            distTo[v] = Double.POSITIVE_INFINITY;
        }
        pq = new IndexMinPQ<>(G.V());
        distTo[0] = 0.0;
        pq.insert(0,0.0);   //用顶点初始化pq
        while (!pq.isEmpty())
            visit(G, pq.delMin());  //将最近的顶点添加到树中
    }

    private void visit(EdgeWeightedGraph G, int v){
        marked[v] = true;
        for (Edge e : G.adj(v)){
            int w = e.other(v);
            if (marked[w]) continue;    //v-w失效
            if (e.weight() < distTo[w]){
                edgeTo[w] = e;
                distTo[w] = e.weight();
                if (pq.contains(w))
                    pq.changeKey(w, distTo[w]);
                else
                    pq.insert(w, distTo[w]);
            }
        }
    }

    public Iterable<Edge> edges() {
        Queue<Edge> mst = new Queue<Edge>();
        for (int v = 0; v < edgeTo.length; v++) {
            Edge e = edgeTo[v];
            if (e != null) {
                mst.enqueue(e);
            }
        }
        return mst;
    }

    public double weight() {
        double weight = 0.0;
        for (Edge e : edges())
            weight += e.weight();
        return weight;
    }
}
```

## Kruskal算法

从一片由V棵单顶点的树构成的森林开始并不断地将两棵树合并（用可以找到的最短边）直到剩下一棵树，就是最小生成树。

把所有边都加入到一个优先队列，每次取出一个，用UF来识别是否会形成环，不会的话连接两个点并把边加入队列。

```java
import Chapter1.Queue;
import Chapter1.UF;
import Sort.MinPQ;

public class KruskalMST {
    private Queue<Edge> mst;
    private double weight;

    public KruskalMST(EdgeWeightedGraph G){
        mst = new Queue<>();
        MinPQ<Edge> pq = new MinPQ<>();
        for (Edge e : G.edges())
            pq.insert(e);
        UF uf = new UF(G.V());

        while (!pq.isEmpty() && mst.size() < G.V() - 1){
            Edge e = pq.delMin();   //从pq得到权重最小的边和它的顶点
            int v = e.either(), w = e.other(v);
            if (uf.connected(v,w))  continue;   //忽略失效的边
            uf.union(v,w);  //合并分量
            mst.enqueue(e); //将边添加到最小生成树
            weight += e.weight();
        }
    }

    public Iterable<Edge> edges(){
        return mst;
    }
    
    public double weight(){
        return weight;
    }
}
```

