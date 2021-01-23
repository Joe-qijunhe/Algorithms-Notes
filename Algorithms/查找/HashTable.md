[toc]

# 散列表

如果所有的键都是小整数，我们可以用一个数组来实现无序的符号表，将键作为数组的索引而数组中键i处储存的就是它对应的值。

散列表的查找算法分两步：

-   用散列函数将被查找的键转化为数组的一个索引
-   处理碰撞冲突

## 正整数

将整数散列最常用的方法是除留余数法。选择大小为素数M的数组，对于任意正整数k，计算k除以M的余数。如果M不是素数，可能导致我们无法均匀地散列散列值。

## 浮点数

将键表示为二进制然后使用除留余数法

## 字符串

Horner算法

# 基于拉链法地散列表

将大小为M的数组中的每个元素指向一条链表，链表中的每个结点都存储了散列值为该元素的索引的键值对。

查找：

-   根据散列值找到对应的链表
-   沿着链表顺序查找相应的键

链表的平均长度时N/M

```java
public class SeparateChainingHashST<Key,Value> {
    private static final int INIT_CAPACITY = 4;

    private int n;  //键值对总数
    private int m;  //散列表大小
    private SequentialSearchST<Key,Value>[] st; //存放链表对象的数组

    public SeparateChainingHashST(){
        this(INIT_CAPACITY);
    }

    public SeparateChainingHashST(int m){
        this.m = m;
        st = (SequentialSearchST<Key, Value>[]) new SequentialSearchST[m];
        for (int i = 0; i < m; i++) {
            st[i] = new SequentialSearchST<>();
        }
    }

    // hash function for keys - returns value between 0 and m-1 (assumes m is a power of 2)
    // (from Java 7 implementation, protects against poor quality hashCode() implementations)
    private int hash(Key key) {
        int h = key.hashCode();
        h ^= (h >>> 20) ^ (h >>> 12) ^ (h >>> 7) ^ (h >>> 4);
        return h & (m-1);
    }

    // hash function for keys - returns value between 0 and m-1
    private int hashTextbook(Key key) {
        return (key.hashCode() & 0x7fffffff) % m;
    }

    public int size(){
        return n;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    public boolean contains(Key key){
        if (key == null) return false;
        return get(key) != null;
    }

    public Value get(Key key){
        if (key == null) return null;
        return st[hash(key)].get(key);
    }

    public void put(Key key, Value val){
        if (key == null) return;
        if (val == null) {
            delete(key);
            return;
        }
        if (n >= 10*m)
            resize(2*m);
        int i = hash(key);
        if (!st[i].contains(key))
            n++;
        st[i].put(key, val);
    }

    public void delete(Key key){
        if (key == null) return;
        int i = hash(key);
        if (st[i].contains(key))
            n--;
        st[i].delete(key);
        if (m > INIT_CAPACITY && n <= 2*m)
            resize(m/2);
    }

    private void resize(int chains) {
        SeparateChainingHashST<Key, Value> temp = new SeparateChainingHashST<Key, Value>(chains);
        for (int i = 0; i < m; i++) {
            for (Key key : st[i].keys()) {
                temp.put(key, st[i].get(key));
            }
        }
        this.m  = temp.m;
        this.n  = temp.n;
        this.st = temp.st;
    }
}

```

# 基于线性探测法的散列表

用大小为M的数组保存N个键值对1，其中M>N。依靠数组中的空位解决碰撞冲突。这叫开发地址散列表。

我们用散列函数找到键在数组中的索引，检查其中的键和被查找的键是否相同。如果不同则继续查找（将索引增大，到达数组结尾时折回数组地开头），直到找到该键或遇到一个空元素。其核心思想是与其将内存用作链表，不如将他们作为在散列表的空元素。这些空元素可以作为查找结束的标志。

```java
package SearchTree;

public class LinearProbingHashST<Key, Value> {
    private static final int INIT_CAPACITY = 4;
    private int n;  //键值对总数
    private int m;  //线性探测表的大小
    private Key[] keys;
    private Value[] vals;

    public LinearProbingHashST() {
        this(INIT_CAPACITY);
    }

    public LinearProbingHashST(int capacity) {
        m = capacity;
        n = 0;
        keys = (Key[]) new Object[capacity];
        vals = (Value[]) new Object[capacity];
    }

    public int size() {
        return n;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    private int hashTextbook(Key key) {
        return (key.hashCode() & 0x7fffffff) % m;
    }

    private int hash(Key key) {
        int h = key.hashCode();
        h ^= (h >>> 20) ^ (h >>> 12) ^ (h >>> 7) ^ (h >>> 4);
        return h & (m - 1);
    }

    public boolean contains(Key key) {
        if (key == null) throw new IllegalArgumentException("argument to contains() is null");
        return get(key) != null;
    }

    public void put(Key key, Value val) {
        if (key == null) return;
        if (val == null) {
            delete(key);
            return;
        }
        if (n >= m / 2) resize(2 * m);
        int i;
        for (i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key)) {
                vals[i] = val;
                return;
            }
        }
        keys[i] = key;
        vals[i] = val;
        n++;
    }

    public Value get(Key key) {
        if (key == null) return null;
        for (int i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key)) {
                return vals[i];
            }
        }
        return null;
    }
}

```

## 删除

删除键时，不能直接将键所在的位置设为null，因为这会使得在此位置之后的元素无法被查找。因此我们需要将簇中被删除键的右侧的所有键重新插入散列表。

```java
public void delete(Key key) {
    if (key == null) return;
    if (!contains(key)) return;

    int i = hash(key);
    while (!key.equals(keys[i]))
        i = (i + 1) % m;
    keys[i] = null;
    vals[i] = null;
    
    i = (i + 1) % m;
    while (keys[i] != null) {
        Key keyToReHash = keys[i];
        Value valToReHash = vals[i];
        keys[i] = null;
        vals[i] = null;
        n--;
        put(keyToReHash, valToReHash);
        i = (i + 1) % m;
    }
    n--;
    
    if (n > 0 && n <= m / 8) resize(m / 2);
}
```

## 调整数组大小

创建一个新的给定大小的LinearProbingHashST（新建一个表是因为哈希值与表的大小有关，扩容后位置不一样，需要重新put），将原表中的所有键重新散列并插入到新表中，再更新原表的keys和vals

```java
private void resize(int capacity) {
    LinearProbingHashST<Key, Value> temp = new LinearProbingHashST<Key, Value>(capacity);
    for (int i = 0; i < m; i++) {
        if (keys[i] != null) {
            temp.put(keys[i], vals[i]);
        }
    }
    keys = temp.keys;
    vals = temp.vals;
    m = temp.m;
}
```