[toc]

# 数据压缩

## 霍夫曼压缩

压缩流程：

-   读取输入
-   将输入中的每个char值的出现频率制成表格
-   根据频率构造霍夫曼编码树
-   构造编码表
-   将单词查找树编码输出
-   将单词总数编码输出
-   使用编码表翻译每个输入字符

### 树结点

```java
   // Huffman trie node
    private static class Node implements Comparable<Node> {
        private final char ch;
        private final int freq;
        private final Node left, right;

        Node(char ch, int freq, Node left, Node right) {
            this.ch    = ch;
            this.freq  = freq;
            this.left  = left;
            this.right = right;
        }

        // is the node a leaf node?
        private boolean isLeaf() {
            return (left == null) && (right == null);
        }

        // compare, based on frequency
        public int compareTo(Node that) {
            return this.freq - that.freq;
        }
    }

```

### 编码树的建造

创建一片由许多只有一个结点（叶子节点）的树组成的森林。接下来自底向上根据频率构造这棵树。构造过程：首先找到两个频率最小的结点，然后创建一个以二者为子结点的新结点（新结点的频率为两个子结点的频率之和）。这个操作会将森林中的树的数量减一。不断重复这个过程，最终所有结点都会被合并为一棵单独的单词查找树。

这样频率较低的结点会被安排在树的底层，高频率的结点会被安排在根结点附近的地方。

```java
    // build the Huffman trie given frequencies
    private static Node buildTrie(int[] freq) {

        // initialze priority queue with singleton trees
        MinPQ<Node> pq = new MinPQ<Node>();
        for (char c = 0; c < R; c++)
            if (freq[c] > 0)
                pq.insert(new Node(c, freq[c], null, null));

        // merge two smallest trees
        while (pq.size() > 1) {
            Node left  = pq.delMin();
            Node right = pq.delMin();
            Node parent = new Node('\0', left.freq + right.freq, left, right);
            pq.insert(parent);
        }
        return pq.delMin();
    }
```

### 构造编译表

编译表将每个字符和它的比特字符串相关联。

```java
    // make a lookup table from symbols and their encodings
    private static void buildCode(String[] st, Node x, String s) {
        if (!x.isLeaf()) {
            buildCode(st, x.left,  s + '0');
            buildCode(st, x.right, s + '1');
        }
        else {
            st[x.ch] = s;
        }
    }
```

### 写入和读取单词树

为了保证数据压缩流程的完整，必须在压缩时将树写入比特流并在展开时候读取他。需要基于单词查找树的前序遍历。内部结点写入0，叶子结点写入1和8位ASCII编码。

```java
    // write bitstring-encoded trie to standard output
    private static void writeTrie(Node x) {
        if (x.isLeaf()) {
            BinaryStdOut.write(true);
            BinaryStdOut.write(x.ch, 8);
            return;
        }
        BinaryStdOut.write(false);
        writeTrie(x.left);
        writeTrie(x.right);
    }
```

```java
    private static Node readTrie() {
        boolean isLeaf = BinaryStdIn.readBoolean();
        if (isLeaf) {
            return new Node(BinaryStdIn.readChar(), -1, null, null);
        }
        else {
            return new Node('\0', -1, readTrie(), readTrie());
        }
    }
```

### 完整压缩过程

```java
public class Huffman {

    // alphabet size of extended ASCII
    private static final int R = 256;

    public static void compress() {
        // read the input
        String s = BinaryStdIn.readString();
        char[] input = s.toCharArray();

        // tabulate frequency counts
        int[] freq = new int[R];
        for (int i = 0; i < input.length; i++)
            freq[input[i]]++;

        // build Huffman trie
        Node root = buildTrie(freq);

        // build code table
        String[] st = new String[R];
        buildCode(st, root, "");

        // print trie for decoder
        writeTrie(root);

        // print number of bytes in original uncompressed message
        BinaryStdOut.write(input.length);

        // use Huffman code to encode input
        for (int i = 0; i < input.length; i++) {
            String code = st[input[i]];
            for (int j = 0; j < code.length(); j++) {
                if (code.charAt(j) == '0') {
                    BinaryStdOut.write(false);
                }
                else if (code.charAt(j) == '1') {
                    BinaryStdOut.write(true);
                }
                else throw new IllegalStateException("Illegal state");
            }
        }

        // close output stream
        BinaryStdOut.close();
    }
}
```

展开被编码过的比特流过程：

-   读取单词查找树
-   读取需要解码的字符数量
-   使用单词查找树解码

```java
    public static void expand() {

        // read in Huffman trie from input stream
        Node root = readTrie(); 

        // number of bytes to write
        int length = BinaryStdIn.readInt();

        // decode using the Huffman trie
        for (int i = 0; i < length; i++) {
            Node x = root;
            while (!x.isLeaf()) {
                boolean bit = BinaryStdIn.readBoolean();
                if (bit) x = x.right;
                else     x = x.left;
            }
            BinaryStdOut.write(x.ch, 8);
        }
        BinaryStdOut.close();
    }
```



