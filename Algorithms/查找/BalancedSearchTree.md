# 2-3查找树

一棵2-3查找树或为一棵空树，或由以下结点组成：

-   2-结点，含有一个键（以及其对应的值）和两条链接，左链接指向的2-3树中的键都小于该结点，右链接指向的2-3树的键都大于该结点
-   3-结点，含有两个键（以及其对应的值）和三条链接，左链接指向的2-3树中的键都小于该结点，中链接指向的2-3树中的键都位于该结点的两个键之间，右链接指向的2-3树的键都大于该结点

# 红黑二叉查找树

红黑二叉查找树背后的基本思想是用标准的二叉查找树（完全由2-结点构成）和一些额外的信息（替换3-结点）。红链接将两个2-结点连接起来构成一个3-结点，黑链接则是2-3树中的普通链接。

红黑树另一种定义是含有红黑链接并满足下列条件的二叉查找树：

-   红链接均为左链接
-   没有任何一个结点同时和两条红链接相连
-   该树是完美黑色平衡的，即任意空链接到根结点的路径上的黑链接数量相同。

## 旋转

左旋转：将两个键中的较小者作为根结点变为较大者作为根结点。

旋转操作都会返回一条链接，这重置父结点相应的链接。h = rotateLeft(h)

```java
private Node rotateLeft(Node h){
    Node x = h.right;
    h.right = x.left;
    x.left = h;
    x.color = h.color;
    h.color = RED;
    x.n = h.n;
    h.n = 1 + size(h.left) + size(h.right);
    return x;
}

private Node rotateRight(Node h){
    Node x = h.left;
    h.left = x.right;
    x.right = h;
    x.color = h.color;
    h.color = RED;
    x.n = h.n;
    h.n = 1 + size(h.left) + size(h.right);
    return x;
}
```

在沿着插入点到根结点的路径向上移动时（通过h.left = put(..)和return h来向上移动）在所经过的每个结点中顺序完成一下操作，我们就能完成插入操作：

1.  如果右子结点是红色的而左子结点是黑色的，左旋转
2.  如果左子结点是红色的且它的左子结点也是红色的，右选择
3.  如果左右子子结点均为红色，颜色转换

第一个if会将任意含有红色右链接的3-结点（或临时4-结点）左旋转，第二条if语句会将临时的4-结点中的两条连续红链接中的上层链接右旋转。第三条if进行颜色转换并将红链接向上传递。

```java
public void put(Key key, Value val) {
    root = put(root, key, val);
    root.color = BLACK;
}

private Node put(Node h, Key key, Value val) {
    if (h == null)
        return new Node(key, val, 1, RED);
    int cmp = key.compareTo(h.key);
    if (cmp < 0)
        h.left = put(h.left, key, val);
    else if (cmp > 0)
        h.right = put(h.right, key, val);
    else
        h.val = val;
    if (isRed(h.right) && !isRed(h.left))
        h = rotateLeft(h);
    if (isRed(h.left) && isRed(h.left.left))
        h = rotateRight(h);
    if (!isRed(h.left) && !isRed(h.right))
        flipColors(h);
    h.n = size(h.left) + size(h.right) + 1;
    return h;
}
```

# 2-3-4树

## 插入

2-3-4中允许存在4-结点。插入算法沿查找路径向下进行变换保证当前结点不是4-结点（这样树底只会遇到2-结点或3-结点，有地方插入），沿查找路径向上进行变换为了将之前创建的4-结点配平。

-   将4-结点表示为由三个2-结点组成的一棵平衡的子树，根结点和两个子结点都用红链接相连
-   在向下的过程中分解所有4-结点并进行颜色转换。如果根结点是4-结点，将他分成三个2结点，树高加一；如果遇到父结点为2结点的4结点，4-结点分解为两个2-结点并将中间键传递给它的父结点，父结点变成3-结点；如果遇到父结点为3-结点的4-结点，4-结点分解为两个2-结点并将中间键传递给它的父结点，父结点变成一个4结点。
-   向上的过程中旋将4结点配平（4结点可以存在，所有允许一个结点同时连接到两条链接）

## 删除

删除最小键：

从2-结点中删除一个键会留下一个空结点，替换成空链接会影响树的完美平衡性。为了保证我们不会删除一个2-结点，沿着左链接向下进行变换，确保当前结点不是2-结点。

根结点有两种情况：

1.  如果根是2-结点且它的两个子结点都是2-结点，直接将这三个结点变成4-结点
2.  保证根结点的左子结点不是2-结点

最后能得到一个含有最小键的3-结点或4-结点，然后将其删除。将3-结点变为2-结点，或者将4-结点变为3-结点，然后再往上分解临时4-结点

```java
    public void deleteMin() {
        if (isEmpty()) throw new NoSuchElementException("BST underflow");

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMin(root);
        if (!isEmpty()) root.color = BLACK;
        // assert check();
    }

    private Node deleteMin(Node h) { 
        if (h.left == null)
            return null;

        if (!isRed(h.left) && !isRed(h.left.left))
            h = moveRedLeft(h);

        h.left = deleteMin(h.left);
        return balance(h);
    }

    private Node moveRedLeft(Node h) {
        //假设结点h为红色，h.left和h.left都是黑色
        //将h.left或h.left的子结点之一变红
        flipColors(h);
        if (isRed(h.right.left)) { 
            h.right = rotateRight(h.right);
            h = rotateLeft(h);
            flipColors(h);
        }
        return h;
    }

    private void flipColors(Node h) {
        h.color = !h.color;
        h.left.color = !h.left.color;
        h.right.color = !h.right.color;
    }

    private Node balance(Node h) {
        if (isRed(h.right) && !isRed(h.left))    h = rotateLeft(h);
        if (isRed(h.left) && isRed(h.left.left)) h = rotateRight(h);
        if (isRed(h.left) && isRed(h.right))     flipColors(h);

        h.size = size(h.left) + size(h.right) + 1;
        return h;
    }
```

```java
    public void deleteMax() {
        if (isEmpty()) throw new NoSuchElementException("BST underflow");

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMax(root);
        if (!isEmpty()) root.color = BLACK;
    }

    private Node deleteMax(Node h) { 
        if (isRed(h.left))
            h = rotateRight(h);

        if (h.right == null)
            return null;

        if (!isRed(h.right) && !isRed(h.right.left))
            h = moveRedRight(h);

        h.right = deleteMax(h.right);

        return balance(h);
    }    
	
	private Node moveRedRight(Node h) {
        //假设结点h为红色，h.right和h.right.left都是黑色
        //将h.right或h.right的子结点之一变红
        flipColors(h);
        if (isRed(h.left.left)) { 
            h = rotateRight(h);
            flipColors(h);
        }
        return h;
    }
```

```java
   public void delete(Key key) { 
        if (key == null) throw new IllegalArgumentException("argument to delete() is null");
        if (!contains(key)) return;

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = delete(root, key);
        if (!isEmpty()) root.color = BLACK;
    }

    private Node delete(Node h, Key key) { 
        if (key.compareTo(h.key) < 0)  {
            if (!isRed(h.left) && !isRed(h.left.left))
                h = moveRedLeft(h);
            h.left = delete(h.left, key);
        }
        else {
            if (isRed(h.left))
                h = rotateRight(h);
            if (key.compareTo(h.key) == 0 && (h.right == null))
                return null;
            if (!isRed(h.right) && !isRed(h.right.left))
                h = moveRedRight(h);
            if (key.compareTo(h.key) == 0) {
                Node x = min(h.right);
                h.key = x.key;
                h.val = x.val;
                h.right = deleteMin(h.right);
            }
            else h.right = delete(h.right, key);
        }
        return balance(h);
    }
```

