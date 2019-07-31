---
layout:     post
title:      "排序算法对比，及时间复杂度剖析"
subtitle:   " \"六种排序算法\""
date:       2019-2-27 12:00:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Algorithm
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## [算法基础-1]—— 排序算法对比，及时间复杂度剖析
> Auther: xuwang </br>
> Mail: xuwang.me@gmail.com </br>
> Created Data: 2019/02/27  &emsp; Modified Data: 2019/02/27
---

#### 1. 排序算法：冒泡、选择、插入、归并、快速、堆排序共六个，这六个都为基于比较的排序。而桶排序(包括计数排序、基数排序)为非基于比较的排序 

1. 冒泡排序（已经被废弃）

时间复杂度: O(N^2)

额外空间复杂度：O(1)

```java
class Solution {
    public void bubbleSort(int[] a){
        if (a == null || a.length < 2){
            return;
        }
        for (int end = a.length-1; end>0; end--){
            for (int i = 0; i < end; i++){
                if (a[i] > a[i+1]){
                    swap(a, i, i+1);
                }
            }
        }
    }
    private void swap(int[] a, int i, int j){
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```

2. 选择排序（已经被废弃）

时间复杂度：O(N^2)

额外空间复杂度：O(1)


```java
class Solution {
    public void selectionSort(int[] a){
        if (a == null || a.length < 2){
            return;
        }
        for (int start = 0; start < a.length-1; start++){
            int minIndex = start;
            for (int j = start; j < a.length; j++){
                minIndex = a[j] < a[minIndex] ? j : minIndex;
            }
            swap(a, start, minIndex);
        }
    }
    private void swap(int[] a, int i, int j){
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```

3. 插入排序（在工程中仍然有使用）

时间复杂度：O(N^2)，==当数据状况未知的情况下，算法复杂度一般按照最差的情况来估计。==

额外空间复杂度：O(1)

==注意：第二个循环中的a[j]>a[j+1]判断写入循环中，可减少循环次数，当前j位置的数比j+1小时，前面的肯定也都满足，故跳出循环==

因此，插入排序的时间复杂度估计分为最好情况，最差情况和平均情况，而选择排序和冒泡排序严格遵守O(N^2)。

- 最好情况：数组已经有序的情况下：O(N)
- 最差情况：数组逆序的情况下：O(N^2)
- 平均/一般情况：

```java
class Solution {
    public void insertSort(int[] a){
        if (a == null || a.length < 2){
            return;
        }
        for (int end = 1; end <= a.length-1; end++){
            for (int j = end-1; j >= 0 && a[j] > a[j+1]; j--){
                swap(a, j, j+1);
            }
        }
    }
    private void swap(int[] a, int i, int j){
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```

4. 归并排序

递归实现

时间复杂度O(NlogN)

计算过程:T(N) = 2T(N/2) + O(N)
利用master公式即可得到时间复杂度为：O(NlogN)

额外空间复杂度O(N)


```java
class Solution {
    public static void mergeSort(int[] a){
        if (a == null || a.length < 2){
            return;
        }
        sortProcess(a, 0, a.length-1);
    }

    private static void sortProcess(int[] a, int L, int R){
        if (L == R) return;

        int mid = L + ((R - L) >> 1);//位运算优先级低，要括起来
        sortProcess(a, L, mid);
        sortProcess(a, mid+1, R);
        merge(a, L, mid, R);

    }
    private static void merge(int[] a, int L, int mid, int R){
        int[] help = new int[R-L+1];
        int i = 0;
        int p1 = L;
        int p2 = mid+1;
        while (p1 <= mid && p2 <= R){
            help[i++] = a[p1] < a[p2] ? a[p1++] : a[p2++];
        }
        //左，右两个数组必有一个已经越界存储完，另一个还没有，下面将未越界的数组的剩余内容全部添加到help最后
        while (p1 <= mid){
            help[i++] = a[p1++];
        }
        while (p2 <= R){
            help[i++] = a[p2++];
        }
        //在原数组上做修改
        for (i = 0; i < help.length; i++){
            a[i+L] = help[i];
        }
    }
}

```

5. 快速排序

时间复杂度O(N*logN)

5.1 经典快排，将数组的最后一个数当成判别数，拆分后也是将各自数组的最后数当成判别数。小于等于判别数的放左边，大于的放右边。

5.2 基于荷兰国旗算法改进后的经典快排：
仍然是将最后一个数当成判别数，但基于荷兰国旗，分成小于，等于，大于三部分，这样等于的就不用再动了，在小于区域和大于区域上递归。


```java
class Solution {
    public static void quickSort(int[] arr, int L, int R){
        if (L < R){
            int[] p = partition(arr, L, R);
            quickSort(arr, L, p[0]-1);
            quickSort(arr, p[1]+1, R);
        }
    }
    private static int[] partition(int[] arr, int L, int R){
        //小于等于less的为小于区，大于等于more的为大于区，当然最后一个位置先除掉
        int less = L-1;
        int more = R;//最后一个数先放那里不管，最后再交换
        int curr = L;
        while (curr < more){
            if (arr[curr] < arr[R]){
                swap(arr, ++less, curr++);
            }else if (arr[curr] > arr[R]){
                swap(arr, --more, curr);//注意此时curr不能++，因为curr右边的区域换过来的数未知，还需要再判断一次
            }else {
                curr++;
            }
        }
        //将最后一个数和当前more交换，交换后more为等于区最后一个数。
        swap(arr, more, R);
        return new int[] {less + 1, more};//返回等于区的左右指针
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

}
```

5.1和5.2快排都是经典快拍，其弊端，当数组本身就有序时，==每次都只选最后一个数==，这样就不存在大于区域（数组本身由小到大）或者不存在小于区域（逆序），这样就需要执行O(N)次partition，而无论是5.1还是5.2的partition复杂度都为O(N)，这样总的时间复杂度就为O(N^2)。即最差时间复杂度为O(N^2)

最差情况：数组已经有序的情况下：O(N^2)
最好情况：每次都二分,需要执行O(logN)次partition，因此为O(NlogN)

因此引入==随机快排==：即每次的判别数都是随机选的。

5.3 随机快排

随机快排的判别数每次都是随机选的，这样复杂度估计就变成了一个概率事件，因此就不存在最差情况和最好情况，这样复杂度的计算就变成计算期望。
随机快排的长期期望时间复杂度为==O(NlogN)==，数学计算得出。


==额外空间复杂度为：O(logN)==，这是由于荷兰国旗问题中每次都需要等于区域的左右两个断点来记住位置，每一层的断点可以复用，但层与层之间都需要各自的，一共logN层。因此需要O(logN)的空间。

logN的空间复杂度也是一个概率期望，因为最差情况为每次都选最后一个点/最前面一个，复杂度为O(N)

==随机快排就加了第一行的随机选一个数进行swap，其他一样==

```java
class Solution {
    public static void quickSort(int[] arr, int L, int R){
        if (L < R){
            //在L,R之间随机找一个数与R位置替换，接下来的代码和之前一样，复用，实现随机快排。
            swap(arr, L + (int)(Math.random()*(R-L+1)), R);

            int[] p = partition(arr, L, R);
            quickSort(arr, L, p[0]-1);
            quickSort(arr, p[1]+1, R);
        }
    }
    private static int[] partition(int[] arr, int L, int R){
        //小于等于less的为小于区，大于等于more的为大于区，当然最后一个位置先除掉
        int less = L-1;
        int more = R;//最后一个数先放那里不管，最后再交换
        int curr = L;
        while (curr < more){
            if (arr[curr] < arr[R]){
                swap(arr, ++less, curr++);
            }else if (arr[curr] > arr[R]){
                swap(arr, --more, curr);//注意此时curr不能++，因为curr右边的区域换过来的数未知，还需要再判断一次
            }else {
                curr++;
            }
        }
        //将最后一个数和当前more交换，交换后more为等于区最后一个数。
        swap(arr, more, R);
        return new int[] {less + 1, more};//返回等于区的左右指针
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```


6. 堆排序

时间复杂度O(NlogN)

额外空间复杂度O(1)

6.1 满二叉树和完全二叉树

满二叉树：叶节点全在最后一层，而且最后一层全是满的。

完全二叉树：上一层全是满的，最后一层的叶节点从左到右的顺序一次填充、

满二叉树是完全二叉树的最满的情况。即满二叉也是完全二叉。

6.2 用数组表示完全二叉树

当前节点坐标为i：
1. 左孩子坐标：2*i+1
2. 右孩子坐标：2*i+2
3. 父节点坐标：(i-1)/2
    以上计算得到的坐标均需要判断不能越界才能使用

6.3 堆：

堆就是完全二叉树

堆分为大根堆和小根堆，又称为最大堆和最小堆：
1. 大根堆：堆中的任何一颗子树的最大值都是这颗子树的头部
2. 小根堆：堆中的任何一颗子树的最小值都是这颗子树的头部

6.4 建立大根堆的过程：

==一次heapInsert的时间复杂度为O(log index)==

因此，建立大根堆的时间复杂度为N-1次循环：
log1 + log2+ log3+log4+log5+...+logN-1，收敛于O(N)
因此建立大根堆的时间复杂度为O(N)


```java
class Solution {
    public static void heapSort(int[] arr, int L, int R){
        for (int i = 1; i < arr.length; i++){
            heapInsert(arr, i);
        }
    }

    //往堆中插入节点，构建堆，每次插入复杂度为log i
    private static void heapInsert(int[] arr, int index){
        while (arr[(index - 1) / 2] < arr[index]){
            swap(arr, (index - 1) / 2, index);
            index = (index - 1) / 2;
        }
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```


6.5 维护大根堆性质。

==一次heapify()的时间复杂度为O(logN)==

```java
class Solution {
    public static void heapSort(int[] arr, int L, int R){
        for (int i = 1; i < arr.length; i++){
            heapInsert(arr, i);
        }
    }

    //往堆中插入节点，构建堆，每次插入复杂度为log i
    private static void heapInsert(int[] arr, int index){
        while (arr[(index - 1) / 2] < arr[index]){
            swap(arr, (index - 1) / 2, index);
            index = (index - 1) / 2;
        }
    }

    //某个值变小后，重新维护大根堆性质
    private static void heapify(int[] arr, int index, int heapSize){
        int left = 2 * index + 1;
        while (left < heapSize){
            int largest = left + 1 < heapSize && arr[left + 1] > arr[left] ? left + 1: left;
            largest = arr[largest] > arr[index] ? largest: index;
            if (largest == index){
                break;
            }
            swap(arr, largest, index);
            index = largest;
            left = index * 2 + 1;
        }
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

}

```


6.6 堆排序：

将大根堆的堆顶和当前堆的最后一个数交换位置，这样当前最大的数就在数组最后一个，然后堆的有效大小减一，维护大根堆性质后再重复。

这样所有当前最大值会依次往后放，最后就成了一个有序堆

时间复杂度O(NlogN)：N次logN的heapify。

额外空间复杂度O(1)

```java
class Solution {
    public static void heapSort(int[] arr){

        if (arr == null || arr.length < 2){
            return;
        }

        //先构建堆
        for (int i = 1; i < arr.length; i++){
            heapInsert(arr, i);
        }
        //堆排序
        int heapSize = arr.length;
        swap(arr, 0, --heapSize);
        while (heapSize > 0){
            heapify(arr, 0, heapSize);
            swap(arr, 0, --heapSize);
        }
    }

    //往堆中插入节点，构建堆，每次插入复杂度为log i
    private static void heapInsert(int[] arr, int index){
        while (arr[(index - 1) / 2] < arr[index]){
            swap(arr, (index - 1) / 2, index);
            index = (index - 1) / 2;
        }
    }

    //某个值变小后，重新维护大根堆性质
    private static void heapify(int[] arr, int index, int heapSize){
        int left = 2 * index + 1;
        while (left < heapSize){
            int largest = left + 1 < heapSize && arr[left + 1] > arr[left] ? left + 1: left;
            largest = arr[largest] > arr[index] ? largest: index;
            if (largest == index){
                break;
            }
            swap(arr, largest, index);
            index = largest;
            left = index * 2 + 1;
        }
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```


#### 2.  递归行为剖析以及时间复杂度计算
递归：将大问题化成小问题求解（分治思想）

递归函数由两部分组成：
1. 终止条件
2. 小问题递归调用，并求解问题。

递归的时间复杂度计算：
1. 第一种情况，master公式的使用
2. 第二种情况，见PPT补充阅读中的特例


#### 3.. 查找算法

1. 二分查找，时间复杂度O(logN)



#### 4. 荷兰国旗问题

荷兰国旗问题（即快排的partition过程）

给定一个数组arr，和一个数num，请把小于num的数放在数组的
左边，等于num的数放在数组的中间，大于num的数放在数组的
右边。

额外空间复杂度O(1)，时间复杂度O(N)

```java
//在L,R之间实现荷兰国旗问题
class Solution {
    public static void partition(int[] arr, int L, int R, int num){
        int less = L-1;
        int more = R+1;
        int curr = L;
        while (curr < more){
            if (arr[curr] < num){
                swap(arr, ++less, curr++);
            }else if (arr[curr] > num){
                swap(arr, --more, curr);//注意此时curr不能++，因为curr右边的区域换过来的数未知，还需要再判断一次
            }else {
                curr++;
            }
        }
    }
    private static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

```


#### 5. 总结


1.. 随机快排和归并排序相比，随机快排更快更具有优势：
    1. 随机快排和归并排序时间复杂度虽然都是O(NlogN)，但归并排序输在常数项（即NlogN的系数上），归并排序每次都需要进行数组拷贝合并，常数项高。
    2. 额外地址空间上，随机快排为O(logN)，归并排序为O(N)。

2. 随机快排在工程上的实现并不是用递归实现的，而是自己进行函数压栈来代替递归的系统函数压栈。

所用的递归都可以改为非递归的形式，自己压栈实现。

3. 堆是一个特别好的数据结构，因为他每次维护堆的性质的复杂度为logN，这是非常好的性质

==优先级队列结构，就是堆结构，在java中PriorityQueue即为堆。==
==在java中TreeMap结构是由红黑树结构实现的==


4. 排序算法的稳定性及汇总：

**排序算法的稳定性：数组中大小相同的值在排序完成之后，在数组中的相对位置不变。**


| 排序算法 | 是否可以做到稳定性                                           |
| -------- | ------------------------------------------------------------ |
| 冒泡排序 | 可以实现，要求在比对过程中，两个数相同时不交换位置即实现了稳定性。 |
| 插入排序 | 可以实现，要求在插入过程中，当两个数相同时停止继续向前比对，即实现了稳定性。 |
| 选择排序 | 无法实现，交换的时候会破坏稳定性                             |
| 归并排序 | 可以实现，在merge过程中，当左边和右边的数值相等时，先拷贝左边的。 |
| 快速排序 | 无法实现，partition的过程无法做到稳定性，实际上可以做到，需要论文级别的难度实现。 |
| 堆排序   | 无法实现，交换的时候会破坏稳定性                             |

**为什么要追求稳定性？**

现实世界中的业务需要做到稳定性，尽可能少的抹去排序之前的原始信息。
姓名，班级，成绩的例子，先按成绩排个序，再按班级排个序，如果实现了稳定性，第二次按照班级排序就不会丢失第一次排序的信息，排序的结果各个班级之内依然是按照成绩排序的。


**工程中的综合排序算法：**
1. 数据类型为基础类型(int, short, long, char等)会调用快排，因为基础类型相同数值大小无差异，不需要稳定性，用快排。
2. 如果是自定义类型会使用归并排序，如定义一个类，当中包含姓名，班级，成绩属性这种。
3. 如果数组长度很短(小于60)，会使用插入排序，因为插排的常数项很低，虽然时间复杂度为O(N^2)，==但所消耗的时间插排仍然有优势==
4. 因此在归并过程中，当递归分治到小于60时，会融合使用插排


6. 非基于比较的排序

桶排序(包括计数排序、基数排序)为非基于比较的排序，非基于比较的排序与被排序的实际数据状况有关系，所以实际中并不经常使用，有性能瓶颈。

时间复杂度O(N)，额外空间复杂度O(N)。

非基于比较的排序是稳定的排序。
