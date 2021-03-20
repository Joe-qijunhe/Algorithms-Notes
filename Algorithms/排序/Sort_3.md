[toc]

# 计数排序

-   找出待排序的数组中最大和最小的元素
-   统计数组中每个值为i的元素出现的次数，存入数组C的第i项
-   对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）
-   反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1

```java
    private int[] countingSort(int[] arr, int maxValue) {
        int bucketLen = maxValue + 1;
        int[] bucket = new int[bucketLen];

        for (int value : arr) {
            bucket[value]++;
        }

        int sortedIndex = 0;
        for (int j = 0; j < bucketLen; j++) {
            while (bucket[j] > 0) {
                arr[sortedIndex++] = j;
                bucket[j]--;
            }
        }
        return arr;
    }
```

# 桶排序

1.  找出待排序数组中的最大值max、最小值min
2.  我们使用 动态数组ArrayList 作为桶，桶里放的元素也用 ArrayList 存储。桶的数量为(max-min)/arr.length+1
3.  遍历数组 arr，计算每个元素 arr[i] 放的桶
4.  每个桶各自排序
5.  遍历桶数组，把排序好的元素放进输出数组

```java
public static void bucketSort(int[] arr){
	
    int max = Integer.MIN_VALUE;
    int min = Integer.MAX_VALUE;
    for(int i = 0; i < arr.length; i++){
        max = Math.max(max, arr[i]);
        min = Math.min(min, arr[i]);
    }
	
    //桶数
    int bucketNum = (max - min) / arr.length + 1;
    ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketNum);
    for(int i = 0; i < bucketNum; i++){
        bucketArr.add(new ArrayList<Integer>());
    }
	
    //将每个元素放入桶
    for(int i = 0; i < arr.length; i++){
        int num = (arr[i] - min) / (arr.length);
        bucketArr.get(num).add(arr[i]);
    }
	
    //对每个桶进行排序
    for(int i = 0; i < bucketArr.size(); i++){
        Collections.sort(bucketArr.get(i));
    }
	
    System.out.println(bucketArr.toString());
	
}
```



# 基数排序

基数排序属于分配式排序，又称桶子法。通过键值的各个位的值，将要排序的元素分配至某些桶中，达到排序作用。

## 原理

将所有待比较数值统一为同样的数位长度，数位较短的数前面补零，然后从最低位开始，依次进行一次排序，这样从最低位排序一直到最高位排序完成后，数列变成一个有序序列。

## 特点

-   基数排序是对传统桶排序的扩展，速度很快
-   经典的空间换时间方式，占用内存很大，当对于海量数据排序时很容易内存不够用
-   基数排序是稳定的
-   有负数的数组不用基数排序来进行排序

```java
    public static void sort(int[] arr) {
        //求出数组中最大数的位数
        int max = arr[0];
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) max = arr[i];
        }
        int maxLength = (max + "").length();

        int[][] bucket = new int[10][arr.length];
        //记录每个桶中实际放了多少数据。bucketElementCounts[0]记录bucket[0]中数据个数
        int[] bucketElementCounts = new int[10];

        for (int i = 0, n = 1; i < maxLength; i++, n *= 10) {
            //针对每个元素的对应位进行排序处理，第一次是个位，第二次是十位。。
            for (int j = 0; j < arr.length; j++) {
                //取出每个元素对应位的值
                int key = arr[j] / n % 10;
                //放入相应的桶
                bucket[key][bucketElementCounts[key]] = arr[j];
                bucketElementCounts[key]++;
            }
            //按照桶里的顺序再放回数组中
            int index = 0;
            for (int k = 0; k < bucketElementCounts.length; k++) {
                if (bucketElementCounts[k] != 0) {
                    for (int l = 0; l < bucketElementCounts[k]; l++)
                        arr[index++] = bucket[k][l];
                }
                //重置
                bucketElementCounts[k] = 0;
            }
        }
    }
```