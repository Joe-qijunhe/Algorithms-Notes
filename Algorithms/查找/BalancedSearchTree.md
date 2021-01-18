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