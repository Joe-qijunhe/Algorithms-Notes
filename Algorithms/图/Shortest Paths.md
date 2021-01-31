[toc]

# 最短路径

>   在一幅加权有向图中，从顶点s到顶点t的最短路径是所有从s到t的路径中的权重最小者。

>   给定一幅加权有向图和一个顶点s，以s为起点的一棵最短路径树（SPT）是图的一幅子图，它包含s和从s可达的所有顶点。这棵有向树的根结点为s，树的每条路径都是有向图中的一条最短路径。

加权有向边

```java
public class DirectedEdge {
    private final int v;    //边的起点
    private final int w;    //边的终点
    private final double weight;    //边的权重

    public DirectedEdge(int v, int w, double weight) {
        this.v = v;
        this.w = w;
        this.weight = weight;
    }

    public double weight() {
        return weight;
    }

    public int from() {
        return v;
    }

    public int to() {
        return w;
    }

    public String toString() {
        return String.format("%d->%d %.2f", v, w, weight);
    }
}
```

加权有向图

```java
import Chapter1.Bag;

public class EdgeWeightedDigraph {
    private final int V;    //顶点总数
    private int E;  //边的总数
    private Bag<DirectedEdge>[] adj;    //邻接表

    public EdgeWeightedDigraph(int V) {
        this.V = V;
        this.E = 0;
        adj = (Bag<DirectedEdge>[]) new Bag[V];
        for (int v = 0; v < V; v++)
            adj[v] = new Bag<>();
    }

    public int V() {
        return V;
    }

    public int E() {
        return E;
    }

    public void addEdge(DirectedEdge e) {
        adj[e.from()].add(e);
        E++;
    }

    public Iterable<DirectedEdge> adj(int v) {
        return adj[v];
    }

    public Iterable<DirectedEdge> edges() {
        Bag<DirectedEdge> bag = new Bag<>();
        for (int v = 0; v < V; v++) {
            for (DirectedEdge e : adj[v])
                bag.add(e);
        }
        return bag;
    }
}
```

## Dijkstra算法

遍历临边时，不用像最小生成树那样检查是否经过那点。最小生成树需要检查是因为不能生成环，而最短路径需要放松边。

Prime里的distTo是最近的一条边的权重，Dijkstra里是起到到这的累计起来的权重