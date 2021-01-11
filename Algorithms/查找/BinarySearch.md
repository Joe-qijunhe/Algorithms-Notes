[toc]
# 顺序查找（基于无序链表）

顺序查找：在查找中一个一个地顺序遍历符号表中的所有键并使用equals()方法来寻找与被查找地键匹配地键。

>   向一个空表中插入N个不同的键需要 $N^2/2$ 次比较

```java
package SearchTree;

import Chapter1.Queue;

public class SequentialSearchST<Key,Value> {
    private Node first;
    private int n;

    private class Node{ //链表结点
        Key key;
        Value value;
        Node next;

        public Node(Key key, Value value, Node next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    public int size(){
        return n;
    }

    public boolean isEmpty(){
        return size() == 0;
    }

    public boolean contains(Key key){
        if (key == null) return false;
        return get(key) != null;
    }

    public Value get(Key key){  //查找给定的值，返回相关的值
        if (key == null) return null;
        for (Node x = first; x != null; x = x.next){
            if (key.equals(x.key)){
                return x.value; //找到
            }
        }
        return null; //未找到
    }

    public void put(Key key, Value value){
        if (key == null) return;
        if (value == null){
            delete(key);
            return;
        }
        for (Node x = first; x != null; x = x.next){
            if (key.equals(x.key)){
                x.value = value;    //找到，更新值
                return;
            }
        }
        first = new Node(key,value,first);  //未找到，新建结点
        n++;
    }

//    public void delete(Key key){
//        if (key == null) return;
//        if (key.equals(first.key)){
//            first = first.next;
//            return;
//        }
//        for (Node x = first; x.next != null; x = x.next){
//            if (key.equals(x.next.key)){
//                x.next = x.next.next;
//                n--;
//                return;
//            }
//        }
//    }
    
    public void delete(Key key) {
        if (key == null) return;
        first = delete(first, key);
    }

    private Node delete(Node x, Key key) {
        if (x == null) return null;
        if (key.equals(x.key)) {
            n--;
            return x.next;
        }
        x.next = delete(x.next, key);
        return x;
    }

    public Iterable<Key> keys()  {
        Queue<Key> queue = new Queue<Key>();
        for (Node x = first; x != null; x = x.next)
            queue.enqueue(x.key);
        return queue;
    }
    
    public static void main(String[] args) {
        SequentialSearchST<String, Integer> st = new SequentialSearchST<>();
        st.put("jojo",1);
        st.put("dio",2);
        st.put("joester",3);
        for (String key : st.keys()){
            System.out.println(key);
        }
    }
}

```

# 二分查找（基于有序数组）

核心是rank()方法，它返回小于给定键的键的数量。告诉我们到哪里去跟新它的值，以及键不在表中时将键存在何处。

-   如果表中存在该键，rank()应该返回该键的位置，也就是表中小于它的键的数量

-   如果表中不存在该键，rank()还是返回表中小于它的键的数量

循环结束时lo的值正好等于表中小于被查找的键的键的数量。因为最后一次循环，lo=mid=hi，key比mid指向的元素小的话，当前就是插入位置；如果key比mid指向的元素大的话，插入位置是当前位置加一。

>   在N个键的有序数组进行二分查找最多需要 (lgN +1) 次比较（无论成功与否）

>   向大小为N的有序数组中插入一个新的元素在最坏情况下，需要访问 2N 次数组，因此向一个空符号表中插入N个元素在最坏情况下需要访问 $N^2$ 次数组。



```java
package SearchTree;

import Chapter1.Queue;

public class BinarySearchST<Key extends Comparable<Key>, Value> {
    private Key[] keys;
    private Value[] values;
    private int n;

    public BinarySearchST(int capacity) {
        keys = (Key[]) new Object[capacity];
        values = (Value[]) new Object[capacity];
    }

    public int size() {
        return n;
    }

    public boolean isEmpty() {
        return n == 0;
    }

    public Value get(Key key) {
        if (isEmpty()) return null;
        int i = rank(key);
        if (i < n && keys[i].compareTo(key) == 0) {
            return values[i];
        } else {
            return null;
        }
    }

    public void put(Key key, Value value) {
        int i = rank(key);
        if (i < n && keys[i].compareTo(key) == 0) {
            values[i] = value;
            return;
        }

        if (n == keys.length) resize(2 * keys.length);

        for (int j = n; j > i; j--) {
            keys[j] = keys[j - 1];
            values[j] = values[j - 1];
        }
        keys[i] = key;
        values[i] = value;
        n++;
    }

    public int rank(Key key) {
        int lo = 0, hi = n - 1;
        while (lo <= hi) {
            int mid = (lo + hi) / 2;
            int cmp = key.compareTo(keys[mid]);
            if (cmp < 0) hi = mid - 1;
            else if (cmp > 0) lo = mid + 1;
            else return mid;
        }
        return lo;
    }

    private void resize(int capacity) {
        Key[] tempk = (Key[]) new Comparable[capacity];
        Value[] tempv = (Value[]) new Object[capacity];
        for (int i = 0; i < n; i++) {
            tempk[i] = keys[i];
            tempv[i] = values[i];
        }
        values = tempv;
        keys = tempk;
    }

    public void delete(Key key) {
        if (isEmpty()) return;
        // compute rank
        int i = rank(key);
        // key not in table
        if (i == n || keys[i].compareTo(key) != 0) {
            return;
        }
        for (int j = i; j < n - 1; j++) {
            keys[j] = keys[j + 1];
            values[j] = values[j + 1];
        }
        n--;
        keys[n] = null;  // to avoid loitering
        values[n] = null;
        // resize if 1/4 full
        if (n > 0 && n == keys.length / 4) resize(keys.length / 2);
    }

    public boolean contains(Key key) {
        if (key == null) return false;
        return get(key) != null;
    }


    public Key min() {
        return keys[0];
    }

    public Key max() {
        return keys[n - 1];
    }

    public void deleteMin() {
        if (isEmpty()) return;
        delete(min());
    }

    public void deleteMax() {
        if (isEmpty()) return;
        delete(max());
    }

    public Key select(int k) {
        if (k < 0 || k >= size()) {
            return null;
        }
        return keys[k];
    }
    
    //如果没找到key，rank返回的位置上的元素大于key，前面一个元素小于key，所以需要i-1
    public Key floor(Key key) {
        if (key == null) return null;
        int i = rank(key);
        if (i < n && key.compareTo(keys[i]) == 0) return keys[i];
        if (i == 0) return null;
        else return keys[i - 1];
    }

    public Key ceiling(Key key) {
        if (key == null) return null;
        int i = rank(key);
        if (i == n) return null;
        else return keys[i];
    }

    public int size(Key lo, Key hi) {
        if (lo == null) return -1;
        if (hi == null) return -1;

        if (lo.compareTo(hi) > 0) return 0;
        if (contains(hi)) return rank(hi) - rank(lo) + 1;
        else return rank(hi) - rank(lo);
    }

    public Iterable<Key> keys() {
        return keys(min(), max());
    }

    public Iterable<Key> keys(Key lo, Key hi) {
        if (lo == null) return null;
        if (hi == null) return null;

        Queue<Key> queue = new Queue<Key>();
        if (lo.compareTo(hi) > 0) return queue;
        for (int i = rank(lo); i < rank(hi); i++)
            queue.enqueue(keys[i]);
        if (contains(hi)) queue.enqueue(keys[rank(hi)]);
        return queue;
    }
}

```

## 二分查找的迭代实现

```java
public static int binarySearch(int key, int[]a){
    int lo = 0;
    int hi = a.length - 1;
    while (lo <= hi){
        int mid = (lo + hi)/2;
        if (key < a[mid]){
            hi = mid -1;
        }
        else if (key > a[mid]){
            lo = mid + 1;
        }
        else{
            return mid;
        }
    }
    return -1;
}
```

## 递归实现
```java
public static int binarySearch2(int key, int[] a, int lo, int hi){
    if (lo > hi){
        return -1;
    }
    int mid = (lo + hi)/2;
    if (key < a[mid]){
        return binarySearch2(key,a,lo,mid-1);
    }else if (key > a[mid]){
        return binarySearch2(key,a,mid+1, hi);
    }else{
        return mid;
    }
}
```

## 找元素第一次出现的索引

主要思想就是找到这个元素时（key == a[mid]），不要停下，继续往前缩小范围，知道lo和hi相等。

```java
public static int findFirst(int key, int[] a) {
    int lo = 0;
    int hi = a.length - 1;

    while (lo < hi) {
        int mid = (lo + hi) / 2;

        if (key <= a[mid]) {
            hi = mid;
        }
        else if (key > a[mid]) {
            lo = mid + 1;
        }
    }

    if (a[lo] == key) {
        return lo;
    } else {
        return -1;
    }
}
```

**注意**：

找最后一次出现不可以用下面这个方法。比如 findLast(2,new int[]{1,2,3})

| lo   | hi   | mid  | a[mid] |
| ---- | ---- | ---- | ------ |
| 0    | 2    | 1    | 2      |
| 1    | 2    | 1    | 2      |
| 1    | 2    | 1    | 2      |

a[mid]一直等于key，会一直执行lo = mid，循环出不去

```java
public static int findLast(int key, int[] a) {
    int lo = 0;
    int hi = a.length - 1;

    while (lo < hi) {
        
        int mid = (lo + hi) / 2;
        if (key < a[mid]) {
            hi = mid - 1;
        }
        else {
            lo = mid;
        }
    }
    if (a[lo] == key) {
        return hi;
    } else {
        return -1;
    }
}
```

## 找左右边界

```java
    public static int search3(int[] nums, int target) {      
        // 搜索右边界，即元素最后一次出现的后面一个索引
        int i = 0, j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] <= target) i = m + 1;
            else j = m - 1;
        }
        int right = i;
        // 若数组中无 target ，则提前返回
        if(j >= 0 && nums[j] != target) return 0;
        
        // 搜索左边界，即元素第一次出现的前面一个索引
        i = 0; j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] < target) i = m + 1;
            else j = m - 1;
        }
        int left = j;
        
        //返回这个元素一共出现了几次。元素出现的次数 = 右边界 - 左边界 - 1
        return right - left - 1;
    }
```

## 稀疏字符串数组

稀疏数组搜索。有个排好序的字符串数组，其中散布着一些空字符串，编写一种方法，找出给定字符串的位置。

 输入：words = ["at", "", "", "", "ball", "", "", "car", "", "","dad", "", ""], s = "ball"

 输出：4

```java
public static int findString(String[] words, String s){
        if(words.length == 1 && words[0].equals(s))
            return 0;
        int i = 0, j = words.length - 1;
        while(i <= j){
            //把i放到第一个非空，把j放到最后一个非空
            while(i < j && words[i].equals("")) i ++;
            while(i < j && words[j].equals("")) j --;
            if(i <= j){
                int mid = (i + j) / 2;
                //用temp往后遍历，找非空，方便将来用mid
                int tmp = mid;
                while(tmp < j && words[tmp].equals("")) tmp ++;
                if(words[tmp].compareTo(s) > 0){
                    j = mid - 1;
                }else if(words[tmp].compareTo(s) < 0){
                    i = tmp + 1;
                }else{
                    return tmp;
                }
            }
        }
        return -1;
    }
```

## 找缺失的数字

排序数组中的搜索问题，首先想到 二分法 解决

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

输入: [0,1,3]

输出: 2

输入: [0,1,2,3,4,5,6,7,9]

输出: 8

```java
public static int missingNumber(int[] nums) {
    int left = 0;
    int right = nums.length-1;
    while (left <= right){
        int mid = (left+right)/2;
       //如果等于index，说明前面都没有缺数字
        if (nums[mid] == mid){
            left = mid + 1;
        }
        //因为数字是连续的，不等于index时，在这个范围里就有缺的值
        else if (nums[mid] > mid){
            right = mid - 1;
        }
    }
    return left;
}
```

