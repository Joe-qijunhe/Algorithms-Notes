[toc]

# 二叉树

## 遍历

前序遍历：先输出父结点，再遍历左子树和右子树

中序遍历：先遍历左子树，再输出父结点，再遍历右子树

后序遍历：先遍历左子树，再遍历右子树，最后输出父结点

看输出父结点的顺序，确定前中后序

```java
class Node<Value> {
    public Value val;
    public Node<Value> left;
    public Node<Value> right;

    public Node(Value val) {
        this.val = val;
    }
}

public class BinaryTree<Value> {
    private Node<Value> root;

    public void preOrder() {
        if (root != null) {
            preOrder(root);
        }
    }

    private void preOrder(Node node) {
        System.out.println(node.val);
        if (node.left != null) {
            preOrder(node.left);
        }
        if (node.right != null) {
            preOrder(node.right);
        }
    }

    public void inOrder() {
        if (root != null) {
            inOrder(root);
        }
    }

    private void inOrder(Node node) {
        if (node.left != null) {
            inOrder(node.left);
        }
        System.out.println(node.val);
        if (node.right != null) {
            inOrder(node.right);
        }
    }

    public void postOrder() {
        if (root != null) {
            postOrder(root);
        }
    }

    private void postOrder(Node node) {
        if (node.left != null) {
            postOrder(node.left);
        }
        if (node.right != null) {
            postOrder(node.right);
        }
        System.out.println(node.val);
    }
}

```

## 查找

前序查找：如果当前结点等于要找的，返回当前结点。否则，如果左子结点不为空，递归前序查找左子节点。如果找到就返回，否者判断右子节点是否为空，查找右子节点

```java
    public Node preOrderSearch(Value value) {
        Node n = preOrderSearch(root, value);
        return n;
    }

    private Node preOrderSearch(Node node, Value value) {
        if (node.val.equals(value)) {
            return node;
        }
        Node retNode = null;
        if (node.left != null) {
            retNode = preOrderSearch(node.left, value);
        }
        if (retNode != null) {
            return retNode;
        }
        if (node.right != null) {
            retNode = preOrderSearch(node.right, value);
        }
        return retNode;
    }

    public Node inOrderSearch(Value value) {
        Node n = inOrderSearch(root, value);
        return n;
    }

    private Node inOrderSearch(Node node, Value value) {
        Node retNode = null;
        if (node.left != null) {
            retNode = inOrderSearch(node.left, value);
        }
        if (retNode != null) {
            return retNode;
        }
        if (node.val.equals(value)) {
            return node;
        }
        if (node.right != null) {
            retNode = inOrderSearch(node.right, value);
        }
        return retNode;
    }

    public Node postOrderSearch(Value value) {
        Node n = postOrderSearch(root, value);
        return n;
    }

    private Node postOrderSearch(Node node, Value value) {
        Node retNode = null;

        if (node.left != null) {
            retNode = postOrderSearch(node.left, value);
        }
        if (retNode != null) {
            return retNode;
        }

        if (node.right != null) {
            retNode = postOrderSearch(node.right, value);
        }
        if (retNode != null) {
            return retNode;
        }
        if (node.val.equals(value)) {
            return node;
        }
        return retNode;
    }
```

