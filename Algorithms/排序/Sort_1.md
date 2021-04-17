[toc]

# 冒泡排序

## 原理

从第一个元素开始，向后依次比较每一对相邻的元素，即R[0]比R[1]，R[1]比R[2] 。若逆序则交换，这样比到最后，最后的元素会是最大的。每一轮都得到一个最大的，所以长度为N的数组需要进行N-1次。

如果一趟比较下来没有进行交换，就说明序列有序，因此要在排序过程中设置以个标志flag判断是否进行过交换，避免不必要的判断。这样，如果用冒泡排序排一个有序的数组（最优情况），只需要比较N-1次就知道是有序的，可以停下了，时间复杂度为O(N)

```java
public class BubbleSort {
	
	public static void bubbleSort(int[] data) {
		int arrayLength = data.length;
		for (int i = 0; i < arrayLength - 1; i++) {
			for (int j = 0; j < arrayLength - 1 - i; j++) {
				if (data[j] > data[j + 1]) {
					int temp = data[j + 1];
					data[j + 1] = data[j];
					data[j] = temp;
				}
			}
		}
	}
	
    //优化，加上flag判断数组是否已经有序，最优情况可以O(N)
	public static void bubbleSort1(int[] data) {
		int arrayLength = data.length;
		for (int i = 0; i < arrayLength - 1; i++) {
			boolean flag = false;
			for (int j = 0; j < arrayLength - 1 - i; j++) {
				if (data[j] > data[j + 1]) {
					int temp = data[j + 1];
					data[j + 1] = data[j];
					data[j] = temp;
					flag = true;
				}
			}
			if (!flag)
				break;
		}
	}

}
```

# 选择排序

## 原理

找出数组中最小的元素，将它和数组的第一个元素交换位置（如果第一个元素就是最小元素那么它就和自己交换）。之后，在剩下的元素中找到最小的元素，将它与数组的第二个元素交换位置。如此反复，直达将整个数组排序。

>   对于长度为N的数组，选择排序需要大约 $\frac{N^2}2$ 次比较 和 N次交换
>
>   说明：
>
>   0到N-1的任意i进行 N-1-i 次比较，因此总共 $(N-1)+(N-2)+...+2+1=\frac{N(N-1)}{2} = \frac{N^2}2$  次比较
>
>   交换元素的代码写在内循环之外，每次交换都能排定一个元素，因此交换的次数是N

## 特点

-   运行时间与输入无关。有序数组和随机排列数组所用的排序时间一样长
-   数据移动是最少的。用了N次交换，交换次数与数组大小是线性关系。

```java
public class Selection {
    public static void sort(Comparable[] a) {
        int N = a.length;
        //这层循环代表着在往第几个位置放最小数
        for (int i = 0; i < N; i++) {
            int min = i;
            //当前元素左边已经有序
            for (int j = i + 1; j < N; j++) {
                if (less(a[j],a[min])){
                    min = j;
                }
            }
            exch(a,i,min);
        }
    }

    public static boolean less(Comparable v, Comparable w) {
        return v.compareTo(w) < 0;
    }

    public static void exch(Comparable[] a, int i, int j) {
        Comparable t = a[i];
        a[i] = a[j];
        a[j] = t;
    }

    public static void show(Comparable[] a) {
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i] + " ");
        }
    }

    public static boolean isSorted(Comparable[] a) {
        for (int i = 1; i < a.length; i++) {
            if (less(a[i], a[i - 1])) {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        Integer[] a = new Integer[]{7,3,5,7,2,6,1,0};
        Selection.sort(a);
        Selection.show(a);
    }
}

```





# 插入排序

## 原理

在待排序的元素中，假设前面n-1(其中n>=2)个数已经是排好顺序的（但最终位置还不确定，为了给更小的元素腾出空间，他们可能会被移动），现将第n个数插到前面已经排好的序列中，然后找到合适自己的位置，使得插入第n个数的这个序列也是排好顺序的。按照此法对所有元素进行插入（索引到达数组的右端）。

## 特点

所需的时间取决于输入中元素的初始顺序，对一个很大且其中的元素以及有序（或接近有序）的数组进行排序将会比对随机顺序的数组或逆序数组进行排序快很多。所以对实际中某些非随机数组很有效。

>   1.  对于随机排列的长度为N且主键不重复的数组，平均情况下插入排序需要$\frac{N^2}{4}$ 次比较以及 $\frac{N^2}{4}$次交换。最坏情况下需要$\frac{N^2}{2}$次比较和$\frac{N^2}{2}$交换，最好情况下需要N-1次比较和0次交换。
>
>       说明：
>
>       在平均情况下每个元素都可能向后移动半个数组的长度。插入排序不会移动比插入的元素更小的元素，即找到插入位置就会停止往下遍历，所以它所需的比较次数平均只有选择排序的一般。
>
>       

>   2.  插入排序需要的交换操作和数组中倒置的数量相同，需要的比较次数大于等于倒置的数量，小于等于倒置的数量加上数组的大小再减一。
>
>       说明：
>
>       每次交换都改变了两个顺序颠倒的元素的位置，相当于减少了一对倒置，当倒置数量为0时，排序就完成了。
>       
>       an inversion is a pair of indices i < j such that A[i] > A[j]. 可以认为插入排序时间复杂度为O(n + I)，n是数组大小，I是倒置的数量。

```java
public class Insertion {
    public static void sort(Comparable[] a) {
        int N = a.length;
        for (int i = 1; i < N; i++) {
            for (int j = i; j > 0 && less(a[j], a[j - 1]); j--){
                exch(a,j,j-1);
            }
        }
    }
}

```

## 题

>   在插入排序找出最小的元素并将其放置于数组的最左边。这样能去掉内循环判断条件j>0。（这是一种常见的规避边界的方法，能够省略判断条件的叫哨兵）。在插入排序中实现使较大元素右移一位只访问一次数组。

```java
public static void sort(Comparable[] a) {
    int n = a.length;

    // put smallest element in position to serve as sentinel
    int exchanges = 0;
    //冒泡排序的一层，确定了数组中最小的一个元素
    for (int i = n-1; i > 0; i--) {
        if (less(a[i], a[i-1])) {
            exch(a, i, i-1);
            exchanges++;
        }
    }
    if (exchanges == 0) return;


    // insertion sort with half-exchanges
    for (int i = 2; i < n; i++) {
        Comparable v = a[i];
        int j = i;
        while (less(v, a[j-1])) {
            //较大元素右移一位
            a[j] = a[j-1];
            j--;
        }
        a[j] = v;
    }

    assert isSorted(a);
}
```



# 希尔排序

## 原理

对于大规模乱序数组插入排序很慢，因为它只会交换相邻的元素，假如主键最小的元素在数组末尾，它只能一点一点地移。希尔排序时直接插入排序的一种更高效的改进版本，交换不相邻的元素以对数组的局部进行排序，并最终用插入排序将局部有序的数组排序。

把较大的数据集合分割成若干个小组（逻辑上分组），然后对每一个小组分别进行插入排序，此时，插入排序所作用的数据量比较小（每一个小组），插入的效率比较高。步长减小，进行插入排序。随着步长的减小，每组的元素越来越多，当步长减为1时，整个数组为一组，插入排序，算法终止。

## 特点

-   任何以1结尾的h序列，我们都能够将数组排序。无最优的递增序列。

-   运行时间达不到平方级别。比插入排序和选择排序快很多，数组越大，优势越大。

## 例子

如图，长度为16的数组初始步长为13，第二轮while，步长h变成了4

![ShellSort](E:\project\Blog\Notes\Algorithms\src\ShellSort.jpg)

1 0 2 3 5 7 2 6 1 0 10 4 9 3 3 7 
1 0 2 3 1 0 2 4 5 3 3 6 9 7 10 7 
0 0 1 1 2 2 3 3 3 4 5 6 7 7 9 10 
0 0 1 1 2 2 3 3 3 4 5 6 7 7 9 10 

方法一

```java
public static void sort(Comparable[] a) {
    int n = a.length;

    int h = 1;
    //求出初始步长，比如长度16的数组，步长为13；长度为10的数组，步长为4。
    //1,4,13,40,121,364...
    while (h < n / 3) {
        h = 3 * h + 1;
    }
    while (h >= 1) {
        for (int i = h; i < n; i++) {
            for (int j = i; j >= h && less(a[j], a[j - h]); j -= h) {
                exch(a, j, j - h);
            }
        }
        h /= 3;
    }
}
```
方法二
```java
public static void shellSort(int[] arr) {
    int length = arr.length;
    int temp;
    for (int step = length / 2; step >= 1; step /= 2) {
        for (int i = step; i < length; i++) {
            temp = arr[i];
            int j = i - step;
            while (j >= 0 && arr[j] > temp) {
                arr[j + step] = arr[j];
                j -= step;
            }
            arr[j + step] = temp;
        }
    }
}
```

