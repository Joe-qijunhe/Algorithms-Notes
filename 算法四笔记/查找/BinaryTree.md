[toc]

# 二叉树

## 遍历

前序遍历：先输出父结点，再遍历左子树和右子树

中序遍历：先遍历左子树，再输出父结点，再遍历右子树

后序遍历：先遍历左子树，再遍历右子树，最后输出父结点

看输出父结点的顺序，确定前中后序

### 递归

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

### 非递归

前序

```java
public void preOrderTraversal(TreeNode root) {
	if (root == null) return;
	Stack<TreeNode> stack = new Stack<TreeNode>(); // 利用栈进行临时存储
	stack.push(root);

	while (!stack.isEmpty()) {
		TreeNode node = stack.pop(); // 取出一个节点，表示开始访问以该节点为根的子树
		visit(node); // 首先访问该节点（先序），之后顺序入栈右子树、左子树
		if (node.right != null) stack.push(node.right);
		if (node.left != null) stack.push(node.left);
	}
}
//或者在节点到达null层时进行判断
public void preOrderTraversal2(TreeNode root) {
	if (root == null) return;
	Stack<TreeNode> stack = new Stack<TreeNode>(); // 利用栈进行临时存储
	TreeNode node = root;
	
	while (!stack.isEmpty() || node != null) { // stack为空且node为null时，说明已经遍历结束
		if (node != null) { // 可以深入左孩子时，先访问，再深入
			visit(node);
			stack.push(node);
			node = node.left;
		} else { // 否则深入栈中节点的右孩子
			node = stack.pop().right;
		}
	}
}
```

中序

```java
//若节点还有左子树，就要先把左子树访问完
//没有左子树可访问时，访问该节点，并尝试访问右子树
public void inOrderTraversal(TreeNode root) {
	if (root == null) return;
	Stack<TreeNode> stack = new Stack<TreeNode>(); // 利用栈进行临时存储
	TreeNode node = root;
	
	while (node != null) { // 当node为null时，说明已经遍历结束
		if (node.left != null) { // 存在左子树时，入栈并深入左子树
			stack.push(node);
			node = node.left;
		} else { // 否则就寻找可以深入右子树的节点
			while (!stack.isEmpty() && node.right == null) {
				// 对于不能深入右子树的节点：直接访问，此时子树访问结束
				visit(node);
				node = stack.pop();
			}
			visit(node); // 如果可以深入右子树，访问该节点后，深入右子树
			node = node.right;
		}
	}
}
//或在节点出栈时访问节点
public void inOrderTraversal2(TreeNode root) {
	if (root == null) return;
	Stack<TreeNode> stack = new Stack<TreeNode>(); // 利用栈进行临时存储
	TreeNode node = root;
	
	while (!stack.isEmpty() || node != null) { // stack为空且node为null时，说明已经遍历结束
		if (node != null) { // 可以深入左孩子
			stack.push(node);
			node = node.left;
		} else { // 否则访问栈中节点，并深入右孩子
			node = stack.pop();
			visit(node);
			node = node.right;
		}
	}
}
```

后序

```java
//尝试按顺序访问该节点的左右子树
//当左右子树都访问完毕时，才可以访问该节点
public void postOrderTraversal(TreeNode root) {
	if (root == null) return;
	Stack<TreeNode> stack = new Stack<TreeNode>(); // 利用栈进行临时存储
	stack.push(root);
	TreeNode lastNode = root; // 为了判断父子节点关系
	
	while (!stack.isEmpty()) {
		TreeNode node = stack.pop(); // 取出一个节点，表示开始访问以该节点为根的子树
		if ((node.left == null && node.right == null) || // 如果该节点为叶子节点
			(node.left == lastNode || node.right == lastNode)) { // 或者已经访问过该节点的子节点
			visit(node); // 直接访问
			lastNode = node;
		} else { // 否则就按顺序把当前节点、右孩子、左孩子入栈
			stack.push(node); 
			if (node.right != null) stack.push(node.right);
			if (node.left != null) stack.push(node.left);
		}
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

## 深度

Depth of a node: number of ancestors not including itself

```java
public int depth(Key target) {
    return depth(root, target, 0);
}

public int depth(Node x, Key target, int level) {
    if (x == null)
        return -1;
    if (x.key.equals(target))
        return level;
    int result = depth(x.left, target, level + 1);
    if (result != -1)
        return result;
    result = depth(x.right, target, level + 1);
    return result;
}
```

## 高度

a 1-node tree has height 0

Height of a tree: maximum depth

```java
public int height() {
    return height(root);
}
private int height(Node x) {
    if (x == null) return -1;
    return 1 + Math.max(height(x.left), height(x.right));
}
```
## 大小

```java
public int size(Node x) {
    if (x == null) {
        return 0;
    }
    return size(x.left) + size(x.right) + 1;
}
```



