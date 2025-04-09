# B树与B+树

# 多路查找树(m-way search tree)

多路查找树是一种用来高效搜索或检索数据的数据结构。m表示最大子节点数。如二叉树也叫2-way查找树。多路查找树主要用来优化搜索(因为树高度更小、每个节点存储的key更多)，最优时间复杂度为O($\log_mn$)。

- 最大子节点数为m
- 节点最大元素数为m-1
- 最大元素数$m^(h+1)-1$,h为树的高度,这里高度定义为树中任一节点到根节点的路径最大值
- 最优时间复杂度为O($\log_mn$),这可以通过使用不同的平衡策略可以达到,如B树：除根节点外的其他非叶子节点最少有m/2个子节点;最差时间复杂度0(n)
- 元素升序排列:左子树值小于根节点,右子树值大于根节点。具体来说：前i个子节点的值均小于第i个key的值;后m-i个子节点的值均大于第i个key的值

以下面的5路查找树为例:

![](/images/Btree/多路查找树.drawio.png)

```java
public class MultiwaySearchTree {

    public static class TreeNode {
        public int val;
        //左子节点
        public TreeNode left;
        //右子节点
        public TreeNode right;
        //下一个节点
        public TreeNode next;

        public TreeNode(int val, TreeNode right, TreeNode left, TreeNode next) {
            this.val = val;
            this.right = right;
            this.left = left;
            this.next = next;
        }

        public TreeNode(int val) {
            this.val = val;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    ", next=" + next +
                    '}';
        }
    }

    public static TreeNode search(TreeNode root, int val) {
        if (root == null) return null;
        if (root.val == val) return root;
        //如果查找值小于当前节点值,以当前节点的左子节点为根继续查找
        if (root.val > val) {
            if (root.left == null) return null;
            return search(root.left, val);
        }
        //如果查找值大于当前节点值且当前节点没有后续节点,以当前节点的右子节点为根继续查找
        if (root.next == null) {
            if (root.right == null) return null;
            return search(root.right, val);
        }
        if (root.next.val == val) return root.next;
        //如果查找值介于当前节点值和当前节点的下一节点值之间,则以当前节点的右子节点为根继续查找
        if (root.next.val < val) return search(root.next, val);
        //如果查找值大于当前节点和当前节点的下一节点值,则以当前节点的下一节点为根继续查找
        return search(root.right, val);
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(18);
        root.next = new TreeNode(54);
        root.next.next = new TreeNode(86);
        root.next.next.next = new TreeNode(400);
        root.next.next.next.right = new TreeNode(450);
        root.next.next.next.right.next = new TreeNode(470);
        root.left = new TreeNode(5);
        root.left.next = new TreeNode(10);
        root.left.right = new TreeNode(7);
        root.left.right.next = new TreeNode(9);
        root.right = new TreeNode(25);
        root.right.next = new TreeNode(35);
        root.right.next.next = new TreeNode(40);
        root.right.next.next.right = new TreeNode(45);
        root.right.left = new TreeNode(19);
        root.right.left.next = new TreeNode(20);
        root.right.left.next.next = new TreeNode(21);
        root.right.left.next.next.next = new TreeNode(22);
        System.out.println(search(root, 40));
    }
}

```

可能以上代码例子还不太直观，不能体现一个节点可以有多个key，下面再看一个例子。

![](/images/Btree/多路查找树2.png)

![](/images/Btree/多路查找树3.png)

相应的数据结构可以表示为

```java
public class Node {
	int count;//子节点数
	int[] value = new int[3];
	Node[] child = new Node[4];
}
```



# B树（B-tree）

B树是一种特殊的多路查找树(B树也是平衡版的二叉树)：B树是自平衡的多路查找树数据结构，能以对数时间搜索，访问，插入和删除元素。B树适用于读写相对大的数据块的存储系统，例如磁盘。B树减少定位记录时所经历的中间过程，从而加快访问速度。B树这种数据结构可以用来描述外部存储。这种数据结构常被应用在数据库和文件系统的实现上。

B树相较于多路查找树有以下两个特性：

- 自平衡：每个叶子节点深度相同
- 除了根节点的节点的元素在[ceil(m/2),m-1]之间(也有说是[m/2-1,m-1])，根节点的元素个数在[1,m-1]之间
- 如果根节点不是叶子节点，那么它至少有两个子节点（2-3树是最简单的B树）；内部节点最少有ceil(m/2)个子节点

插入、删除和搜索操作时间复杂度为O($logn$)，n为树中存储key的个数。另外其空间复杂度为O(n)。

> 辅助存储设备通常容量大但读写慢，因此需要像B树这样的数据结构来减少磁盘访问。其他的数据结构像二叉查找树，avl树，红黑树等的每个节点仅存储一个值，如果需要存储大量值，树都高度将会变大导致访问时间增加。
>
> B树能在一个节点存储多个值并且有多个子节点，这减少了树都高度，从而让磁盘访问更快。

## 插入

插入过程如下:

- 1.如果树为空,构建一个根节点并插入值
- 2.寻找合适的节点插入(符合B树特性的节点)
- 3.如果节点没有多余的位置，按以下步骤执行：
  - 3.1按升序排列插入元素
  - 3.2当前节点已超过最大key数量限制,按中间值分裂
  - 3.3向上弹出中间值,中间值左边的值作为左子节点,中间值右边的值作为右子节点
  - 3.4如果上层节点的值未满,则按升序插入到上层节点
  - 3.5如果上层节点的值已满,重复步骤3

例:构建一个B树，M=3，值为12,23,6,8,15,19,45,1,4,7,5。

![](/images/Btree/B树插入2.drawio.png)



## 删除

### 目标key在叶子节点

- 1.1叶子节点key数量大于最小key数量
  - 1.1.1直接删除目标key

- 1.2叶子节点只有最小数量key
  - 1.2.1从当前左兄弟节点借一个最大key(仅当左兄弟节点key数量大于最小值),将该key移到父节点,将父节点key移到目标节点以替换要删除的目标key。如无法执行上述动作，执行1.2.2
  - 1.2.2从当前右兄弟节点借一个最小key(仅当右兄弟节点key数量大于最小值),将该key移到父节点,将父节点key移到目标节点以替换要删除的目标key。如无法执行上述动作，执行1.2.3
  - 1.2.3与左兄弟节点和父节点key合并或者与右兄弟节点和父节点key合并,然后删除目标key

以M=5的一颗B树来演示上述步骤：

![' '](/images/Btree/B树删除-叶子节点.drawio.png)

### 目标key在内部节点

- 2.1有序前驱(左子树最大元素)
  - 2.1.1找到左子树中最大key代替要删除的目标key。左子树元素数量要大于节点最小元素数量
- 2.2有序后继(右子树最小元素)
  - 2.2.1找到右子树中最小key代替要删除的目标key。右子树元素数量要大于节点最小元素数量
- 2.3上述都不满足,合并左右子节点和目标key,然后删除目标key

![](/images/Btree/B树删除-内部节点.drawio.png)

### 高度减少

无法从左右子树取值时，合并导致高度减少。

![](/images/Btree/B树高度减少.drawio.png)

以下Java代码来自[java-algorithms-implementation仓库](https://github.com/phishman3579/java-algorithms-implementation/blob/master/src/com/jwetherell/algorithms/data_structures/BTree.java)

```java
package datastructure;

import java.util.*;

/**
 * @author cheysen
 * @date 2025/4/3 12:36
 * @description B 树是一种树数据结构，它使数据保持排序并允许对数时间的搜索、顺序访问、插入和删除。
 * B 树是二叉搜索树的泛化，因为一个节点可以有两个以上的子节点。与自平衡二叉搜索树不同，B 树针对读取和写入大型数据块的系统进行了优化。它通常用于数据库和文件系统。
 **/
@SuppressWarnings("unchecked")
public class BTree<T extends Comparable<T>> implements Tree<T> {
    //2-3树是最简单的B树:最大key数量2,最大子节点数3,最小key数量2
    private int minKeySize = 1;
    private int minChildrenSize = minKeySize + 1;
    private int maxKeySize = 2 * minKeySize;
    private int maxChildrenSize = maxKeySize + 1;

    private Node<T> root = null;
    private int size = 0;

    //默认2-3树
    public BTree() {

    }

    //order:非根节点最少key数量
    public BTree(int order) {
        this.minKeySize = order;
        this.minChildrenSize = minKeySize + 1;
        this.maxKeySize = 2 * minKeySize;
        this.maxChildrenSize = maxKeySize + 1;
    }

    public BTree<T> appendAdd(T value) throws Exception {
        boolean added = add(value);
        if (added) {
            return this;
        } else {
            throw new Exception("add " + value + " error");
        }
    }

    @Override
    public boolean add(T value) {
        if (root == null) {
            root = new Node<T>(null, maxKeySize, maxChildrenSize);
            root.addKey(value);
        } else {
            Node<T> node = root;
            while (node != null) {
                //没有子节点就在当前节点插入
                if (node.numberOfChildren() == 0) {
                    node.addKey(value);
                    if (node.numberOfKeys() <= maxKeySize) {
                        break;
                    }
                    //key数量已满，需要分裂
                    split(node);
                    break;
                }
                //否则，按有序的规则查找该插入的节点
                T lesser = node.getKey(0);
                if (value.compareTo(lesser) <= 0) {
                    node = node.getChild(0);
                    continue;
                }
                int numberOfKeys = node.numberOfKeys();
                int last = numberOfKeys - 1;
                T greater = node.getKey(last);
                if (value.compareTo(greater) > 0) {
                    node = node.getChild(numberOfKeys);
                    continue;
                }
                for (int i = 1; i < node.numberOfKeys(); i++) {
                    T prev = node.getKey(i - 1);
                    T next = node.getKey(i);
                    if (value.compareTo(prev) > 0 && value.compareTo(next) <= 0) {
                        node = node.getChild(i);
                        break;
                    }
                }
            }
        }
        size++;
        return true;
    }

    @Override
    public T remove(T value) {
        T removed = null;
        Node<T> node = this.getNode(value);
        removed = remove(value, node);
        return removed;
    }

    @Override
    public boolean contains(T value) {
        Node<T> node = getNode(value);
        return node != null;
    }

    @Override
    public void clear() {
        root = null;
        size = 0;
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public boolean validate() {
        if (root == null) {
            return true;
        }
        return validateNode(root);
    }

    @Override
    public Collection<T> toCollection() {
        return new JavaCompatibleBTree<>(this);
    }

    @Override
    public String toString() {
        return TreePrinter.getString(this);
    }

    /**
     * 获取目标值所在节点
     * @param value 要查找的值
     * @return 目标值所在节点
     */
    private Node<T> getNode(T value) {
        Node<T> node = root;
        while (node != null) {
            //尝试是否比节点最小值小
            T lesser = node.getKey(0);
            if (value.compareTo(lesser) < 0) {
                if (node.numberOfChildren() > 0) {
                    node = node.getChild(0);
                } else {
                    node = null;
                }
                continue;
            }
            //尝试是否比节点最大值大
            int numberOfKeys = node.numberOfKeys();
            int lastIndex = numberOfKeys - 1;
            T greater = node.getKey(lastIndex);
            if (value.compareTo(greater) > 0) {
                if (node.numberOfChildren() > numberOfKeys) {
                    node = node.getChild(numberOfKeys);
                } else {
                    node = null;
                }
                continue;
            }
            //不满足极大极小情况,在中间值中比较
            for (int i = 0; i < numberOfKeys; i++) {
                T currentValue = node.getKey(i);
                if (currentValue.compareTo(value) == 0) {
                    return node;
                }
                int next = i + 1;
                if (next <= lastIndex) {
                    T nextValue = node.getKey(next);
                    //目标值介于第i与第i+1个key之间时,在第i+1个子节点(如果有)中继续查找
                    if (value.compareTo(currentValue) > 0 && value.compareTo(nextValue) < 0) {
                        if (next < node.numberOfChildren()) {
                            node = node.getChild(next);
                            break;
                        }
                        return null;
                    }
                }
            }
        }
        return null;
    }

    /**
     * 从节点中删除值
     * @param value 要删除的值
     * @param node 要删除的值所在节点
     * @return true-删除成功
     */
    private T remove(T value, Node<T> node) {
        if (node == null) {
            return null;
        }
        T removed = null;
        int index = node.indexOf(value);
        removed = node.removeKey(index);
        //叶子节点元素的删除:1.直接删除 2.兄弟节点借值 3.合并兄弟节点(之一)与父节点key
        if (node.numberOfChildren() == 0) {
            if (node.parent != null && node.numberOfKeys() < minKeySize) {
                this.combined(node);
            } else if (node.parent == null && node.numberOfKeys() == 0) {
                //只有一个元素根节点删除,清空树
                root = null;
            }
        } else {
            //内部节点元素的删除:1.有序前驱值替换 2.有序后继值替换 3.合并目标值的左右子节点
            Node<T> lesser = node.getChild(index);
            //找到最大有序前驱
            Node<T> greatestNode = this.getGreatestNode(lesser);
            T replaceValue = this.removeGreatestValue(greatestNode);
            node.addKey(replaceValue);
            //借最大有序前驱相当于在子节点中删除一个最大值
            if (greatestNode.parent != null && greatestNode.numberOfKeys() < minKeySize) {
                this.combined(greatestNode);
            }
            //combined中不需考虑分裂的情况:借值不用考虑;合并只在左右兄弟节点key数量等于minKeySize时,合并后最大为maxKeySize
            //而这里是先借值,再合并,如果是借最有序前驱,假如右兄弟节点有maxKeySize,合并后就需要分裂
            if (greatestNode.numberOfChildren() > maxChildrenSize) {
                this.split(greatestNode);
            }
        }
        size--;
        return removed;
    }

    /** 当节点key数量不足最小数量时组合父节点和子节点key.组合包括向兄弟节点借值和合并两种情况
     * @param node 组合的子节点
     * @return true-组合成功
     */
    private boolean combined(Node<T> node) {
        Node<T> parent = node.parent;
        int indexOfParentChild = parent.indexOf(node);
        int indexOfLeftNeighbor = indexOfParentChild - 1;
        int indexOfRightNeighbor = indexOfParentChild + 1;
        Node<T> rightNeighbor = null;
        //这里只是初始化
        int rightNeighborSize = -minChildrenSize;
        //如果有右兄弟节点
        if (indexOfRightNeighbor < parent.numberOfChildren()) {
            rightNeighbor = parent.getChild(indexOfRightNeighbor);
            rightNeighborSize = rightNeighbor.numberOfKeys();
        }

        //尝试向兄弟节点借值(左右顺序没有先后)
        if (rightNeighbor != null && rightNeighborSize > minKeySize) {
            //右兄弟节点的最小值
            T removeValue = rightNeighbor.getKey(0);
            //父节点中有序前驱(相对于给定值而言)的索引
            int prev = getIndexOfPreviousValue(parent, removeValue);
            T parentValue = parent.removeKey(prev);
            T neighborValue = rightNeighbor.removeKey(0);
            node.addKey(parentValue);
            parent.addKey(neighborValue);
            if (rightNeighbor.numberOfChildren() > 0) {
                node.addChild(rightNeighbor.removeChild(0));
            }
        } else {
            Node<T> leftNeighbor = null;
            int leftNeighborSize = -minChildrenSize;
            if (indexOfLeftNeighbor >= 0) {
                leftNeighbor = parent.getChild(indexOfLeftNeighbor);
                leftNeighborSize = leftNeighbor.numberOfKeys();
            }
            if (leftNeighbor != null && leftNeighborSize > minKeySize) {
                //左兄弟节点的最大值
                T removeValue = leftNeighbor.getKey(leftNeighbor.numberOfKeys() - 1);
                int nextV = getIndexOfNextValue(parent, removeValue);
                T parentValue = parent.removeKey(nextV);
                T neighborValue = leftNeighbor.removeKey(leftNeighbor.numberOfKeys() - 1);
                node.addKey(parentValue);
                parent.addKey(neighborValue);
                if (leftNeighbor.numberOfChildren() > 0) {
                    node.addChild(leftNeighbor.removeChild(leftNeighbor.numberOfChildren() - 1));
                }

            } else if (rightNeighbor != null && parent.numberOfKeys() > 0) {
                //不能借值,尝试与右兄弟节点合并(左右顺序没有先后)
                T removeValue = rightNeighbor.getKey(0);
                int prev = getIndexOfPreviousValue(parent, removeValue);
                T parentValue = parent.removeKey(prev);
                parent.removeChild(rightNeighbor);
                node.addKey(parentValue);
                for (int i = 0; i < rightNeighbor.keySize; i++) {
                    T v = rightNeighbor.getKey(i);
                    node.addKey(v);
                }
                //非叶子节点合并需要考虑
                for (int i = 0; i < rightNeighbor.childrenSize; i++) {
                    Node<T> c = rightNeighbor.getChild(i);
                    node.addChild(c);
                }

                if (parent.parent != null && parent.numberOfKeys() < minKeySize) {
                    //合并使得父节点key数量不足最小数量,继续向上合并
                    this.combined(parent);
                } else if (parent.numberOfKeys() == 0) {
                    //合并使得父节点没有key,树都高度减少
                    node.parent = null;
                    root = node;
                }
            } else if (leftNeighbor != null && parent.numberOfKeys() > 0) {
                T removeValue = leftNeighbor.getKey(leftNeighbor.numberOfKeys() - 1);
                int nextV = getIndexOfNextValue(parent, removeValue);
                T parentValue = parent.removeKey(nextV);
                parent.removeChild(leftNeighbor);
                node.addKey(parentValue);
                for (int i = 0; i < leftNeighbor.keySize; i++) {
                    T v = leftNeighbor.getKey(i);
                    node.addKey(v);
                }
                for (int i = 0; i < leftNeighbor.childrenSize; i++) {
                    Node<T> c = leftNeighbor.getChild(i);
                    node.addChild(c);
                }
                if (parent.parent != null && parent.numberOfKeys() < minKeySize) {
                    this.combined(parent);
                } else if (parent.numberOfKeys() == 0) {
                    node.parent = null;
                    root = node;
                }
            }
        }
        return true;
    }

    /**
     * 节点key数量大于最大key数量，从中间分裂
     * @param nodeToSplit 要分裂的节点
     */
    private void split(Node<T> nodeToSplit) {
        Node<T> node = nodeToSplit;
        int numberOfKeys = node.numberOfKeys();
        int medianIndex = numberOfKeys / 2;//向下取整
        T medianValue = node.getKey(medianIndex);

        Node<T> left = new Node<>(null, maxKeySize, maxChildrenSize);
        for (int i = 0; i < medianIndex; i++) {
            left.addKey(node.getKey(i));
        }
        //如果中间值是第i个key,那么前i个子节点归为分裂出的左节点的子节点
        if (node.numberOfChildren() > 0) {
            for (int j = 0; j <= medianIndex ; j++) {
                Node<T> c = node.getChild(j);
                left.addChild(c);
            }
        }

        Node<T> right = new Node<>(null, maxKeySize, maxChildrenSize);
        for (int i = medianIndex + 1; i < numberOfKeys ; i++) {
            right.addKey(node.getKey(i));
        }
        //如果中间值是第i个key,那么后m-i个子节点归为分裂出的右节点的子节点
        if (node.numberOfChildren() > 0) {
            for (int j = medianIndex + 1; j < node.numberOfChildren(); j++) {
                Node<T> c = node.getChild(j);
                right.addChild(c);
            }
        }

        //将中间值向上弹出至父节点
        if (node.parent == null) {
            //如果是根节点的分裂,树的高度增加
            Node<T> newRoot = new Node<>(null, maxKeySize, maxChildrenSize);
            newRoot.addKey(medianValue);
            node.parent = newRoot;
            root = newRoot;
            root.addChild(left);
            root.addChild(right);
        } else {
            Node<T> parent = node.parent;
            parent.addKey(medianValue);
            parent.removeChild(node);
            parent.addChild(left);
            parent.addChild(right);
            //如果父节点key数量已满,则继续分裂
            if (parent.numberOfKeys() > maxKeySize) {
                split(parent);
            }
        }

    }

    /**
     * 在节点中找第一个小于给定值的值的索引
     * @param node
     * @param value
     * @return 有序前驱的索引 | -1
     */
    private int getIndexOfPreviousValue(Node<T> node, T value) {
        //只有1个值时,直接返回了
        for (int i = 1; i < node.numberOfKeys(); i++) {
            T t = node.getKey(i);
            if (t.compareTo(value) >= 0) {
                return i - 1;
            }
        }
        return node.numberOfKeys() - 1;
    }

    /**
     * 在节点中第一个大于给定值的索引
     * @param node
     * @param value
     * @return 有序后继的索引 | -1
     */
    private int getIndexOfNextValue(Node<T> node, T value) {
        for (int i = 0; i < node.numberOfKeys(); i++) {
            T t = node.getKey(i);
            if (t.compareTo(value) >= 0) {
                return i;
            }
        }
        return node.numberOfKeys() - 1;
    }

    /**
     * 找到给定节点树下的最大(最右)节点
     * @param nodeToGet
     * @return
     */
    private Node<T> getGreatestNode(Node<T> nodeToGet) {
        Node<T> node = nodeToGet;
        while (node.numberOfChildren() > 0) {
            node = node.getChild(node.numberOfChildren() - 1);
        }
        return node;
    }

    /**
     * 删除节点中的最大值
     * @param node
     * @return
     */
    private T removeGreatestValue(Node<T> node) {
        T value = null;
        if (node.numberOfKeys() > 0) {
            value = node.removeKey(node.numberOfKeys() - 1);
        }
        return value;
    }

    /**
     * 以给定节点为根节点验证B树是否合法
     * @param node 
     * @return
     */
    private boolean validateNode(Node<T> node) {
        int keySize = node.numberOfKeys();
        if (keySize > 1) {
            //验证元素是否升序
            for (int i = 1; i < keySize; i++) {
                T p = node.getKey(i - 1);
                T n = node.getKey(i);
                if (p.compareTo(n) > 0) {
                    return false;
                }
            }
        }
        int childrenSize = node.numberOfChildren();
        if (node.parent == null) {
            //根节点
            if (keySize > maxKeySize) {
                //根节点没有最小元素限制，空树也视为合法
                return false;
            } else if (childrenSize == 0) {
                return true;
            } else if (childrenSize < 2) {
                //2-3树是最简单的B树
                return false;
            } else if (childrenSize > maxChildrenSize) {
                return false;
            }
        } else {
            //非根节点
            if (keySize < minKeySize) {
                return false;
            } else if (keySize > maxKeySize) {
                return false;
            } else if (childrenSize == 0) {
                return true;
            } else if (keySize != (childrenSize - 1)) {
                //key数量可以由子节点树推导，但子节点树不一定是keySize+1
                return false;
            } else if (childrenSize > maxChildrenSize) {
                return false;
            } else if (childrenSize < minChildrenSize) {
                return false;
            }
        }

        //第一个子节点的有序性
        Node<T> first = node.getChild(0);
        if (first.getKey(first.numberOfKeys() - 1).compareTo(node.getKey(0)) > 0) {
            return false;
        }
        //最后一个子节点的有序性
        Node<T> last = node.getChild(node.numberOfChildren() - 1);
        if (last.getKey(0).compareTo(node.getKey(node.numberOfKeys() - 1)) < 0) {
            return false;
        }
        //中间节点的有序性
        for (int i = 1; i < node.numberOfKeys(); i++) {
            T p = node.getKey(i - 1);
            T n = node.getKey(i);
            Node<T> c = node.getChild(i);
            if (p.compareTo(c.getKey(0)) > 0) {
                return false;
            }
            if (n.compareTo(c.getKey(c.numberOfKeys() - 1)) < 0) {
                return false;
            }
        }

        //递归向下验证
        for (int i = 0; i < node.childrenSize; i++) {
            Node<T> c = node.getChild(i);
            boolean valid = this.validateNode(c);
            if (!valid) {
                return false;
            }
        }
        return true;
    }

    private static class Node<T extends Comparable<T>> {
        private T[] keys = null;
        private int keySize = 0;
        private Node<T>[] children = null;
        private int childrenSize = 0;
        private Node<T> parent = null;

        private Node(Node<T> parent, int maxKeySize, int maxChildrenSize) {
            this.parent = parent;
            //+1是为了应对分裂
            this.keys = (T[])new Comparable[maxKeySize + 1];
            this.children = new Node[maxChildrenSize + 1];
            this.keySize = 0;
            this.childrenSize = 0;
        }


        private Comparator<Node<T>> comparator = new Comparator<Node<T>>() {
            @Override
            public int compare(Node<T> o1, Node<T> o2) {
                //TODO 这里该如何定义?
                return o1.getKey(0).compareTo(o2.getKey(0));
            }
        };


        private T getKey(int index) {
            return keys[index];
        }

        private int indexOf(T value) {
            for (int i = 0; i < keySize; i++) {
                if (keys[i].equals(value)) {
                    return i;
                }
            }
            return -1;
        }

        private void addKey(T value) {
            //不使用排序
            int i = 0;
            /*通过向后复制来实现插入,如[11,15,20,0,0]插入17
              [11,15,20,20,0] -> [11,15,17,20,0]
             */
            for (i = keySize - 1; i >=0 && value.compareTo(keys[i]) < 0 ; i--) {
                keys[i + 1] = keys[i];
            }
            keys[i + 1] = value;
            keySize++;
            //使用排序
            /*
             keys[keysSize++] = value;
             Arrays.sort(keys, 0, keysSize);
             */
        }

        //此方法只考虑节点内部元素的删除
        private T removeKey(T value) {
            T removed = null;
            boolean found = false;
            if (keySize == 0) {
                return null;
            }
            for (int i = 0; i < keySize; i++) {
                if (keys[i].equals(value)) {
                    found = true;
                    removed = keys[i];
                } else if (found) {
                    //向前复制删除,第一次循环不会进入这段逻辑,所以i-1>0
                    keys[i -1 ] = keys[i];
                }
            }
            //删除相当于整体向前移一位
            if (found) {
                keySize--;
                keys[keySize] = null;
            }
            return null;
        }

        //此方法只考虑节点内部元素的删除
        private T removeKey(int index) {
            if (index >= keySize) {
                return null;
            }
            T value = keys[index];
            for (int i = index + 1; i < keySize; i++) {
                keys[i - 1] = keys[i];
            }
            keySize--;
            keys[keySize] = null;
            return value;
        }

        private int numberOfKeys() {
            return keySize;
        }

        private int numberOfChildren() {return childrenSize;}

        private Node<T> getChild(int index) {
            if (index >= childrenSize) {
                return null;
            }
            return children[index];
        }

        private int indexOf(Node<T> child) {
            for (int i = 0; i < childrenSize; i++) {
                if (children[i].equals(child)) {
                    return i;
                }
            }
            return -1;
        }

        private boolean addChild(Node<T> child) {
            child.parent = this;
            children[childrenSize++] = child;
            Arrays.sort(children, 0, childrenSize, comparator);
            return true;
        }

        private boolean removeChild(Node<T> child) {
            boolean found = false;
            if (childrenSize == 0) {
                return found;
            }
            for (int i = 0; i < childrenSize; i++) {
                if (children[i].equals(child)) {
                    found = true;
                } else if(found) {
                    children[i - 1] = children[i];
                }
            }
            if (found) {
                childrenSize--;
                children[childrenSize] = null;
            }
            return found;
        }

        private Node<T> removeChild(int index) {
            if (index >= childrenSize) {
                return null;
            }
            Node<T> value = children[index];
            children[index] = null;
            for (int i = index + 1; i < childrenSize; i++) {
                children[i - 1] = children[i];
            }
            childrenSize--;
            children[childrenSize] = null;
            return value;
        }

        @Override
        public String toString() {
            StringBuilder builder = new StringBuilder();
            builder.append("keys=[");
            for (int i = 0; i < numberOfKeys(); i++) {
                T value = getKey(i);
                builder.append(value);
                if (i < numberOfKeys() - 1) {
                    builder.append(", ");
                }
            }
            builder.append("]\n");
            if (parent != null) {
                builder.append("parent=[");
                for (int i = 0; i < parent.numberOfKeys(); i++) {
                    T value = parent.getKey(i);
                    builder.append(value);
                    if (i < parent.numberOfKeys() - 1) {
                        builder.append(", ");
                    }
                builder.append(", ");
                }
            }
            if (children != null) {
                builder.append("keySize=")
                        .append(numberOfKeys())
                        .append(" children=[")
                        .append(numberOfChildren()).append("\n");
            }
            return builder.toString();
        }
    }

    public static class JavaCompatibleBTree<T extends Comparable<T>> extends AbstractCollection<T> {

        private BTree<T> tree = null;

        public JavaCompatibleBTree(BTree<T> tree) {
            this.tree = tree;
        }

        @Override
        public boolean add(T value) {
            return tree.add(value);
        }

        @Override
        public boolean remove(Object value) {
            return tree.remove((T) value) != null;
        }

        @Override
        public boolean contains(Object value) {
            return tree.contains((T) value);
        }

        @Override
        public int size() {
            return tree.size();
        }

        @Override
        public Iterator<T> iterator() {
            return new BTreeIterator<>(this.tree);
        }

        private static class BTreeIterator<C extends Comparable<C>> implements Iterator<C> {

            private BTree<C> tree = null;
            private BTree.Node<C> lastNode = null;
            private C lastValue = null;
            private int index = 0;
            private Deque<BTree.Node<C>> toVisit = new ArrayDeque<>();


            public BTreeIterator(BTree<C> tree) {
                this.tree = tree;
                if (tree.root != null && tree.root.keySize > 0) {
                    toVisit.add(tree.root);
                }
            }

            @Override
            public boolean hasNext() {
                if ((lastNode != null && index < lastNode.keySize) || toVisit.size() > 0) {
                    return true;
                }
                return false;
            }

            @Override
            public C next() {
                if (lastNode != null && (index < lastNode.keySize)) {
                    lastValue = lastNode.getKey(index++);
                    return lastValue;
                }
                if (toVisit.size() > 0) {
                    Node<C> n = toVisit.pop();
                    for (int i = 0; i < n.childrenSize; i++) {
                        toVisit.add(n.getChild(i));
                    }
                    index = 0;
                    lastNode = n;
                    lastValue = lastNode.getKey(index++);
                    return lastValue;
                }
                return null;
            }

            @Override
            public void remove() {
                if (lastNode != null && lastValue != null) {
                    //删除时，重置迭代器（效率很低）
                    tree.remove(lastValue, lastNode);

                    lastNode = null;
                    lastValue = null;
                    index = 0;
                    toVisit.clear();
                    if (tree.root != null && tree.root.keySize > 0) {
                        toVisit.add(tree.root);
                    }
                }
            }
        }

    }

    private static class TreePrinter {
        public static <T extends Comparable<T>> String getString(BTree<T> tree) {
            if (tree.root == null) {
                return "Tree is empty";
            }
            return getString(tree.root, "", true);
        }

        private static <T extends Comparable<T>> String getString(Node<T> node, String prefix, boolean isTail) {
            StringBuilder builder = new StringBuilder();
            builder.append(prefix).append(isTail ? "└── " : "├── ");
            for (int i = 0; i < node.numberOfKeys(); i++) {
                T value = node.getKey(i);
                builder.append(value);
                if (i < node.numberOfKeys() - 1) {
                    builder.append(", ");
                }
            }
            builder.append("\n");
            if (node.children != null) {
                for (int i = 0; i < node.numberOfChildren() - 1; i++) {
                    Node<T> obj = node.getChild(i);
                    builder.append(getString(obj, prefix + (isTail ? "   " : "|   "), false));
                }
                if (node.numberOfChildren() >=1 ) {
                    Node<T> obj = node.getChild(node.numberOfChildren() - 1);
                    builder.append(getString(obj, prefix + (isTail ? "   " : "|   "), true));
                }
            }
            return builder.toString();
        }
    }

    public static void main(String[] args) throws Exception {
        BTree<Integer> tbTree = new BTree<>(1);
        tbTree.add(12);
        tbTree.add(23);
        tbTree.add(6);
        tbTree.add(8);
        tbTree.add(15);
        tbTree.add(19);
        tbTree.add(45);
        tbTree.add(1);
        tbTree.add(4);
        tbTree.add(7);
        tbTree.add(5);
        System.out.println(tbTree);
        tbTree.toCollection().forEach(System.out::println);

        BTree<Integer> fiveWay = new BTree<>(2);
        fiveWay.appendAdd(78)
                .appendAdd(22)
                .appendAdd(75)
                .appendAdd(89)
                .appendAdd(94)
                .appendAdd(10)
                .appendAdd(12)
                .appendAdd(23)
                .appendAdd(31)
                .appendAdd(76)
                .appendAdd(77)
                .appendAdd(86)
                .appendAdd(87)
                .appendAdd(88)
                .appendAdd(90)
                .appendAdd(93)
                .appendAdd(98)
                .appendAdd(99);

        System.out.println(fiveWay);
        fiveWay.remove(22);
        System.out.println(fiveWay);
        fiveWay.toCollection().forEach(System.out::println);
    }
}
```

# B+树

B+树是B树的变体，针对读取大量数据（例如数据库和文件系统）的系统进行了优化。由于在管理大量数据时有更少的I/O次数，所以B+树广泛应用于数据库索引。

- 所有的数据存储在叶子节点，内部节点只存储key：这些key用于将搜索引导到存储实际数据值的适当叶子节点中
- 叶子节点存储key和数据，这些叶子节点互相链接，高效执行范围查询和顺序查询

B树和B+树都差异如下表(表中"节点包含数据"在数据库系统中通常意味着指向数据记录的指针)：

| B树                            | B+树                                          |
| ------------------------------ | --------------------------------------------- |
| 每个节点包含key和数据          | 内部节点仅包含key,叶子节点包含key和数据       |
| \                              | 叶子节点以链表形式链接                        |
| 随机访问更快                   | 范围查询更快,允许顺序访问                     |
| 内部节点存储数据降低空间利用率 | 节点可以存储更多的key,更高效                  |
| 无重复的key                    | key可以重复,内部节点出现的key也存在于叶子节点 |
| 高度更高                       | 高度更小,意味着I/O次数更少                    |

## 插入

B+树都插入根B树类似,需要注意以下两点

- 内部节点分裂时,key不重复
- 子节点分裂时,分裂的中间值在右子节点重复

![](/images/Btree/B+树插入.drawio.png)

## 删除

### key仅在叶子节点

**节点key数量大于最小key数量**

直接删除

**节点只有最小key数量**

删除该key，从直接兄弟节点借一个key，将兄弟节点的中间key移到父节点

![](/images/Btree/B+树删除5.png)

### key存在于内部节点和叶子节点

**key数量大于最小key数量**

直接删除内部节点和叶子节点的key，用有序后继填充内部节点删除的位置

![](/images/Btree/B+树删除45.png)

**只有最小key数量且key所在位置相邻**

删除该key,从直接兄弟节点借一个值，填充内部节点删除的位置

![](/images/Btree/B+树删除35.png)

**只有最小key数量且key所在位置不相邻**

删除该key,合并兄弟节点，用有序后续填充内部节点删除的位置

![](/images/Btree/B+树删除25.png)

### 高度减少

![](/images/Btree/B+树删除高度减少.png)

