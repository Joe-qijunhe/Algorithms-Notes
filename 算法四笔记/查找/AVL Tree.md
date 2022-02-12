[toc]

# 平衡二叉树

平衡二叉树（平衡二叉查找树/AVL树）。它是一棵空树或它的左右两个子树的高度差绝对值不超过1，左右两个子树都是一棵平衡二叉树。常用的实现有红黑树，AVL，替罪羊树等

```java
public class AVLTree<T extends Comparable<T>> {
    public AVLTreeNode<T> root;

    class AVLTreeNode<T extends Comparable<T>> {
        T key;
        int height;
        AVLTreeNode<T> left;
        AVLTreeNode<T> right;

        public AVLTreeNode(T key) {
            this.key = key;
        }
    }

    public int height() {
        return height(root);
    }

    private int height(AVLTreeNode<T> tree) {
        if (tree == null) return -1;
        return tree.height;
    }
    
    private AVLTreeNode<T> minimum(AVLTreeNode<T> tree) {
        if (tree == null)
            return null;

        while (tree.left != null)
            tree = tree.left;
        return tree;
    }

    public T minimum() {
        AVLTreeNode<T> p = minimum(root);
        if (p != null)
            return p.key;

        return null;
    }

    private AVLTreeNode<T> maximum(AVLTreeNode<T> tree) {
        if (tree == null)
            return null;

        while (tree.right != null)
            tree = tree.right;
        return tree;
    }

    public T maximum() {
        AVLTreeNode<T> p = maximum(root);
        if (p != null)
            return p.key;

        return null;
    }
    //...
}
```

## 旋转

旋转分为四种情况：

LL：LeftLeft，也称为”左左”。插入或删除一个节点后，根节点的左子树的左子树还有非空子节点，导致”根的左子树的高度”比”根的右子树的高度”大2，导致AVL树失去了平衡。



![image-20210320164411839](..\src\AVL\ll.png)

```java
    private AVLTreeNode<T> leftLeftRotation(AVLTreeNode<T> k2) {
        AVLTreeNode<T> k1;

        k1 = k2.left;
        k2.left = k1.right;
        k1.right = k2;

        k2.height = Math.max(height(k2.left), height(k2.right)) + 1;
        k1.height = Math.max(height(k1.left), k2.height) + 1;

        return k1;
    }
```

LR：LeftRight，也称为”左右”。插入或删除一个节点后，根节点的左子树的右子树还有非空子节点，导致”根的左子树的高度”比”根的右子树的高度”大2，导致AVL树失去了平衡。（k1还是平衡的，k3检测到不平衡时，如果只用左旋或者右旋是解决不了问题的，因为会把k2挂到k3的左边，还是不平衡。所以需要旋转两次）

![image-20210320164448217](..\src\AVL\lr.png)

```java
    private AVLTreeNode<T> leftRightRotation(AVLTreeNode<T> k3) {
        k3.left = rightRightRotation(k3.left);
        return leftLeftRotation(k3);
    }
```

RR：RightRight，称为”右右”。插入或删除一个节点后，根节点的右子树的右子树还有非空子节点，导致”根的右子树的高度”比”根的左子树的高度”大2，导致AVL树失去了平衡。

![image-20210320164548129](..\src\AVL\rr.png)

```java
    private AVLTreeNode<T> rightRightRotation(AVLTreeNode<T> k1) {
        AVLTreeNode<T> k2;

        k2 = k1.right;
        k1.right = k2.left;
        k2.left = k1;

        k1.height = Math.max(height(k1.left), height(k1.right)) + 1;
        k2.height = Math.max(height(k2.right), k1.height) + 1;

        return k2;
    }
```

RR：RightRight，称为”右右”。插入或删除一个节点后，根节点的右子树的右子树还有非空子节点，导致”根的右子树的高度”比”根的左子树的高度”大2，导致AVL树失去了平衡。

![image-20210320164520236](..\src\AVL\rl.png)

```java
    private AVLTreeNode<T> rightLeftRotation(AVLTreeNode<T> k1) {
        k1.right = leftLeftRotation(k1.right);
        return rightRightRotation(k1);
    }
```

## 插入

插入一个节点可能会破坏AVL树的平衡，可以通过旋转操作来进行修正。

```java
    private AVLTreeNode<T> insert(AVLTreeNode<T> tree, T key) {
        if (tree == null) {
            return new AVLTreeNode<T>(key);
        } else {
            int cmp = key.compareTo(tree.key);

            if (cmp < 0) {    
                tree.left = insert(tree.left, key);
                // 插入节点后，若AVL树失去平衡，则进行相应的调节。
                if (height(tree.left) - height(tree.right) == 2) {
                    //插入的结点在左子树的左边
                    if (key.compareTo(tree.left.key) < 0)
                        tree = leftLeftRotation(tree);
                        //插入的结点在左子树的右边
                    else
                        tree = leftRightRotation(tree);
                }
            } else if (cmp > 0) {    
                tree.right = insert(tree.right, key);
                // 插入节点后，若AVL树失去平衡，则进行相应的调节。
                if (height(tree.right) - height(tree.left) == 2) {
                    //插入的结点在右子树的右边
                    if (key.compareTo(tree.right.key) > 0)
                        tree = rightRightRotation(tree);
                        //插入的结点在右子树的左边
                    else
                        tree = rightLeftRotation(tree);
                }
            } else {    // cmp==0
                System.out.println("添加失败：不允许添加相同的节点！");
            }
        }
        tree.height = Math.max(height(tree.left), height(tree.right)) + 1;
        return tree;
    }

    public void insert(T key) {
        root = insert(root, key);
    }
```

## 删除

1.  当前节点为要删除的节点且是树叶（无子树），直接删除，当前节点（为None）的平衡不受影响。

2.  当前节点为要删除的节点且只有一个左儿子或右儿子，用左儿子或右儿子代替当前节点，当前节点的平衡不受影响。

3.  当前节点为要删除的节点且有左子树右子树:如果右子树高度较高，则从右子树选取最小节点，将其值赋予当前节点，然后删除右子树的最小节点。如果左子树高度较高，则从左子树选取最大节点，将其值赋予当前节点，然后删除左子树的最大节点。这样操作当前节点的平衡不会被破坏。

4.  当前节点不是要删除的节点，则对其左子树或者右子树进行递归操作。当前节点的平衡条件可能会被破坏，需要进行平衡操作。

```java
    private AVLTreeNode<T> remove(AVLTreeNode<T> tree, T key) {
        if (tree == null) return null;
        int cmp = key.compareTo(tree.key);
        if (cmp < 0) {       
            tree.left = remove(tree.left, key);
            // 删除节点后，若AVL树失去平衡，则进行相应的调节。
            if (height(tree.right) - height(tree.left) == 2) {
                AVLTreeNode<T> r = tree.right;
                if (height(r.left) > height(r.right))
                    tree = rightLeftRotation(tree);
                else
                    tree = rightRightRotation(tree);
            }
        } else if (cmp > 0) {  
            tree.right = remove(tree.right, key);
            // 删除节点后，若AVL树失去平衡，则进行相应的调节。
            if (height(tree.left) - height(tree.right) == 2) {
                AVLTreeNode<T> l = tree.left;
                if (height(l.right) > height(l.left))
                    tree = leftRightRotation(tree);
                else
                    tree = leftLeftRotation(tree);
            }
        } else {    // tree是对应要删除的节点。
            // tree的左右孩子都非空
            if ((tree.left != null) && (tree.right != null)) {
                if (height(tree.left) > height(tree.right)) {
                    AVLTreeNode<T> max = maximum(tree.left);
                    tree.key = max.key;
                    tree.left = remove(tree.left, max.key);
                } else {
                    AVLTreeNode<T> min = minimum(tree.right);
                    tree.key = min.key;
                    tree.right = remove(tree.right, min.key);
                }
            } else {
                if (tree.right != null) return tree.right;
                else return tree.left;
            }
        }

        tree.height = Math.max(height(tree.left), height(tree.right)) + 1;
        return tree;
    }

    public void remove(T key) {
        if (key == null) return;
        root = remove(root, key);
    }
```



## 判断是否为平衡二叉树

>   输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

法一：自顶而下判断每个结点的左右子树高度差

```java
class Solution {
    HashMap<TreeNode, Integer> m = new HashMap<>();
    
    public boolean isBalanced(TreeNode root) {
        if (root == null) return true;
        if (Math.abs(height(root.left) - height(root.right)) > 1)
            return false;
        return isBalanced(root.left) && isBalanced(root.right); 
    }
    
    public int height(TreeNode x) {
        if (x == null) return -1;
        if (m.containsKey(x)) return m.get(x);
        int depth = Math.max(height(x.right), height(x.left)) + 1;
        m.put(x,depth);
        return depth;
    }
}
```

法二：自底向上

对二叉树做后序遍历，从底至顶返回子树深度，若判定某子树不是平衡树则 “剪枝” ，直接向上返回-1。

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        return recur(root) != -1;
    }

    private int recur(TreeNode root) {
        if (root == null) return 0;
        int left = recur(root.left);
        if(left == -1) return -1;
        int right = recur(root.right);
        if(right == -1) return -1;
        return Math.abs(left - right) < 2 ? Math.max(left, right) + 1 : -1;
    }
}
```



