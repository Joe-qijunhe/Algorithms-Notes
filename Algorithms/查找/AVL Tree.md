[toc]

# 平衡二叉树

平衡二叉树（平衡二叉查找树/AVL树）。它是一棵空树或它的左右两个子树的高度差绝对值不超过1，左右两个子树都是一棵平衡二叉树。常用的实现有红黑树，AVL，替罪羊树等

旋转分为四种情况：

1.  LL：LeftLeft，也称为”左左”。插入或删除一个节点后，根节点的左子树的左子树还有非空子节点，导致”根的左子树的高度”比”根的右子树的高度”大2，导致AVL树失去了平衡。
    ![image-20210320164411839](E:\project\Blog\Notes\Algorithms\src\ll.png)

2.  LR：LeftRight，也称为”左右”。插入或删除一个节点后，根节点的左子树的右子树还有非空子节点，导致”根的左子树的高度”比”根的右子树的高度”大2，导致AVL树失去了平衡。（k1还是平衡的，k3检测到不平衡时，如果只用左旋或者右旋是解决不了问题的，因为会把k2挂到k3的左边，还是不平衡。所以需要旋转两次）
    ![image-20210320164448217](E:\project\Blog\Notes\Algorithms\src\lr.png)

3.  RL：RightLeft，称为”右左”。插入或删除一个节点后，根节点的右子树的左子树还有非空子节点，导致”根的右子树的高度”比”根的左子树的高度”大2，导致AVL树失去了平衡。
    ![image-20210320164520236](E:\project\Blog\Notes\Algorithms\src\rl.png)

4.  RR：RightRight，称为”右右”。插入或删除一个节点后，根节点的右子树的右子树还有非空子节点，导致”根的右子树的高度”比”根的左子树的高度”大2，导致AVL树失去了平衡。
    ![image-20210320164548129](E:\project\Blog\Notes\Algorithms\src\rr.png)

```java
package SearchTree;

public class AVL<T extends Comparable<T>> {
    private static final int MAX_HEIGHT_DIFFERENCE = 1;
    private Node root;

    private class Node {
        T key;
        int height;
        Node left;
        Node right;

        public Node(T key, Node left, Node right) {
            this.key = key;
            this.left = left;
            this.right = right;
            this.height = 0;
        }
    }

    public int height() {
        return height(root);
    }

    private int height(Node node) {
        if (node != null)
            return node.height;

        return 0;
    }

    public void insert(T key) {
        if (key == null) {
            throw new NullPointerException();
        }
        root = insert(root, key);
    }

    private Node insert(Node node, T key) {
        if (node == null) {
            return new Node(key, null, null);
        }

        int cmp = key.compareTo(node.key);
        if (cmp == 0) {
            return node;
        }
        if (cmp < 0) {
            node.left = insert(node.left, key);
        } else {
            node.right = insert(node.right, key);
        }

        if (Math.abs(height(node.left) - height(node.right)) > MAX_HEIGHT_DIFFERENCE) {
            node = balance(node);
        }
        node.height = Math.max(height(node.left), height(node.right)) + 1;
        return node;
    }


    private Node balance(Node node) {
        Node node1, node2;
        if (height(node.left) > height(node.right)) {
            // ll
            if (height(node.left.left) > height(node.left.right)) {
                node1 = node.left;
                node.left = node1.right;
                node1.right = node;

                node.height = Math.max(height(node.left), height(node.right)) + 1;
                node1.height = Math.max(height(node1.left), height(node1.right)) + 1;
                return node1;
            }
            // lr
            if (height(node.left.right) > height(node.left.left)) {
                node1 = node.left;
                node2 = node.left.right;
                node.left = node2.right;
                node1.right = node2.left;
                node2.left = node1;
                node2.right = node;

                node.height = Math.max(height(node.left), height(node.right)) + 1;
                node1.height = Math.max(height(node1.left), height(node1.right)) + 1;
                node2.height = Math.max(height(node2.left), height(node2.right)) + 1;
                return node2;
            }
        }

        if (height(node.right) > height(node.left)) {
            // rr
            if (height(node.right.right) > height(node.right.left)) {
                node1 = node.right;
                node.right = node1.left;
                node1.left = node;

                node.height = Math.max(height(node.left), height(node.right)) + 1;
                node1.height = Math.max(height(node1.right), node.height) + 1;
                return node1;
            }

            // rl
            if (height(node.right.left) > height(node.right.right)) {
                node1 = node.right;
                node2 = node.right.left;
                node.right = node2.left;
                node1.left = node2.right;
                node2.left = node;
                node2.right = node1;

                node.height = Math.max(height(node.left), height(node.right)) + 1;
                node1.height = Math.max(height(node1.left), height(node1.right)) + 1;
                node2.height = Math.max(height(node2.left), height(node2.right)) + 1;
                return node2;
            }
        }
        return node;
    }

    public void preOrder(Node node) {
        System.out.println(node.key);
        if (node.left != null) {
            preOrder(node.left);
        }
        if (node.right != null) {
            preOrder(node.right);
        }
    }
}

```

删除就是在原来查找二叉树的基础上，每一次删除都调用balance方法平衡AVL树

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

