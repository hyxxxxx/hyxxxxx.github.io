---
layout: post
title: "数据结构及算法——排序"
date: 2017-08-06
excerpt: "主要介绍几个基本排序算法"
tags: 
- Original
- Java
comments: true
---

基本排序算法主要有两大类型：比较类排序和非比较类排序。我们平时说的最多的冒泡啊快速啊都是属于比较类排序，顾名思义，需要元素之间互相比较才能确定位置的排序就是比较排序。

# 比较类排序

这里有几个**通用方法**会用在比较类排序中，下面就不再赘述

1. 比较元素v是否小于元素w

        public static boolean less(Comparable v, Comparable w) {
            return v.compareTo(w) < 0;
        }

2. 交换两个元素的位置

        public static void swap(Comparable[] a, int i, int j) {
            Comparable t = a[i];
            a[i] = a[j];
            a[j] = t;
        }

3. 判断该数组是否有序

        public static boolean isSort(Comparable[] a) {
            for (int i = 1; i < a.length; i++)
                if (less(a[i], a[i - 1]))
                    return false;
            return true;
        }

4. 打算数组顺序
        
        public static void shuffle(Comparable[] a) {
            Random random = new Random();
            for (int i = a.length; i > 1; i--)
                swap(a, i - 1, random.nextInt(i));
        }


## 选择排序
首先，找到数组中最小的那个元素，其次，将它和数组的第一个元素交换位置（如果第一个元素就是最小元素那么它就和自己交换）。再次，在剩下的元素中找到最小的元素，将它与数组的第二个元素交换位置。如此往复，直到将整个数组排序。

**代码实现**

    public static void sort(Comparable[] a) {
        int N = a.length;
        for (int i = 0; i < N; i++) {
            int min = i;		//min用来记录最小值的下标
            for (int j = i + 1; j < N; j++) {
                if (less(a[j], a[min]))
                    min = j;		//如果有比a[min]更小的值就把这个值的下标赋值给min
                /*最后交换第一个元素（i）和min的值，如果最小值是第一个元素本身，就交换它本身*/
                swap(a, i, min);
            }
        }
    }

这种排序**运行时长和元素排列无关**，也就是说一个已有序的数组和一个乱序的数组排序所用的时间一样长；但是这种排序的数据移动是最少的，它交换元素的次数是随着元素数量增多呈线性增长。

## 插入排序
插入排序就是将一个数据插入到已经排好序的有序数据中，从而得到一个新的、个数加一的有序数据。

比如有数组 [4,2,3,5,1]

2比4小所以和4交换位置[4,**2**,3,5,1]

[2,4,3,5,1]

3继续和4交换位置，然后和2比较比2大，则确定了3的位置[2,4,**3**,5,1]

[2,3,4,5,1]

元素1向前面每一个元素比较，比5小交换，比4小交换，比3小交换，比2小交换

最终[1,2,3,4,5]

**代码实现**

    public static void sort(Comparable[] a) {
        int N = a.length;
        for (int i = 1; i < N; i++) {
            for (int j = i; j > 0 && less(a[j], a[j - 1]); j--) {
                swap(a, j, j - 1);      //只要小于前一个元素就交换位置
            }
        }
    }
    
插入排序对部分有序的数组十分有效，也很适合小规模数组。


## 希尔排序
这是一种基于插入排序的排序算法。

希尔排序是把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

希尔排序的思想是是数组中任意间隔为h的元素都是有序的。

**代码实现**

    public static void sort(Comparable[] a) {
        int N = a.length;
        int h = 1;
        while (h < N / 3)   
            h = 3 * h;
        while (h >= 1) {
            //对于每次的分组使用插入排序
            for (int i = h; i < N; i++) {
                for (int j = i; j >= h && less(a[j], a[j - h]); j -= h) {
                    swap(a, j, j - h);
                }
            }
            h = h / 3;
        }
    }
    
希尔排序会比选择和插入排序的性能要高并且**不需要使用额外的内存空间**，它在大型数组的表现也比较好，当然，还有比希尔排序更加高效的算法。

## 归并排序
要将一个大数组排序，可以先（递归地）将它分成两半分别排序，然后将结果归并起来。
归并排序是算法设计中分治思想的典型应用。

### 自顶向下的归并排序
这种归并是把数组分成两半，分别排好序后再归并在一起

**代码实现**

    private static Comparable[] aux;    //定义一个新数组，用于将每次切分的元素在此排序

    public static void sort(Comparable[] a) {
        aux = new Comparable[a.length];
        sort(a, 0, a.length - 1);
    }

    private static void sort(Comparable[] a, int lo, int hi) {
        if (hi <= lo)
            return;
        int mid = lo + (hi - lo) / 2;
        sort(a, lo, mid);       //将左半边排序
        sort(a, mid + 1, hi);   //将右半边排序
        merge(a, lo, mid, hi);  //归并结果
    }

    private static void merge(Comparable[] a, int lo, int mid, int hi) {
        int i = lo, j = mid + 1;
        System.arraycopy(a, lo, aux, lo, hi + 1 - lo);
        for (int k = lo; k <= hi; k++) {    //归并到数组a
            if (i > mid)
                a[k] = aux[j++];
            else if (j > hi)
                a[k] = aux[i++];
            else if (less(aux[j], aux[i]))
                a[k] = aux[j++];
            else
                a[k] = aux[i++];
        }
    }
    
归并排序优点是能保证将任意长度为N的数组排序所需时间和NlogN成正比；它的主要缺点是所需的**额外空间**和N成正比。

### 自底向上的归并排序
这是另一种归并方法，就是先归并那些微型数组，然后再成对归并得到的子数组，如此，知道我们将整个数组归并在一起。

**代码实现**

    public static void sort(Comparable[] a) {
        int N = a.length;
        aux = new Comparable[N];
        for (int sz = 1; sz < N; sz = sz + sz) {
            for (int lo = 0; lo < N - sz; lo += sz + sz) {
                merge(a, lo, lo + sz - 1, Math.min(lo + sz + sz - 1, N - 1));
            }
        }
    }
   
这种方式更适合用**链表组织的数据**，因为只需要重新组织链表链接就可以将链表原地排序。

## 快速排序
快速排序也一种分治的排序算法。

基本思想是通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

简单说来就是每次选择数组中一个值为参照值，把小于参照值的数都放到左边去，大于参照值的数都放到右边去，从而参照值的下标就能确定了，递归地调用这种方法，直到数组有序。

**代码实现**

    public static void sort(Comparable[] a) {
        shuffle(a);     //消除对原有顺序的依赖
        sort(a, 0, a.length - 1);
    }

    private static void sort(Comparable[] a, int lo, int hi) {
        if (hi <= lo)
            return;
        int j = partition(a, lo, hi);   //将数组切分
        sort(a, lo, j - 1);     //将左半部分a[lo..j-1]排序
        sort(a, j + 1, hi);     //将右半部分a[j+1..hi]排序
    }

    private static int partition(Comparable[] a, int lo, int hi) {
        int i = lo, j = hi + 1;
        Comparable v = a[lo];
        while (true) {
            //分别扫描左右，若i>=j则扫描结束
            while (less(a[++i], v))
                if (i == hi)
                    break;
            while (less(v, a[--j]))
                if (j == lo)
                    break;
            if (i >= j)
                break;
            swap(a, i, j);
        }
        swap(a, lo, j);     //将参照值放入确定好的位置
        return j;   //j就是参照值的下标位置
    }
    
这种方式的缺点是不能很好地解决重复值问题。

之所以先将数组打乱，是因为快速排序具有随机性，打乱顺序是保证对所有的子数组都一视同仁。

值得肯定的是这种方式比归并排序要快，因为它移动数据的次数更少。

### 改进快速排序
对于小数组，快速排序比插入排序慢，所以简单改进下就是将`if (hi <= lo) return;`改为`if (hi <= lo + M) { //在这里调用插入排序  return; };`

还有一种改进方法是使用子数组的一小部分元素的中位数来切分数组，这样做得到的切分更好，但代价是需要计算中位数。

**代码实现**

    public static void sort(Comparable[] a, int lo, int hi) {
        if (hi <= lo)
            return;
        int lt = lo, i = lo + 1, gt = hi;
        Comparable v = a[lo];
        while (i <= gt) {
            int cmp = a[i].compareTo(v);
            if (cmp < 0)
                swap(a, lt++, i++);
            else if (cmp > 0)
                swap(a, i, gt--);
            else
                i++;
        }
        sort(a, lo, lt - 1);
        sort(a, gt + 1, hi);
    }
    
这种排序代码的切分能够将和切分元素相等的元素归位，这样它们就不会被包含在递归调用处理的子数组中。对于存在**大量重复元素的数组**，这种方法比标准的快速排序要快得多。

# 非比较类排序

未完待续...

## 桶排序


## 计数排序


## 基数排序


## 堆排序

参考资料：[《算法》（第4版）](https://book.douban.com/subject/19952400/ "《算法》（第4版）")
