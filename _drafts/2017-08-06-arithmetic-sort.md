---
layout: post
title: "数据结构及算法——排序"
date: 2017-07-22
excerpt: "主要介绍几个基本算法"
life: true
tags: 
- Original
- Java
comments: true
---

基本排序算法主要有两大类型：比较类排序和非比较类排序。我们平时说的最多的冒泡啊快速啊都是属于比较类排序，顾名思义，需要元素之间互相比较才能确定位置的排序就是比较排序。

### 比较类排序

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


#### 选择排序
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

#### 插入排序
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


#### 希尔排序


#### 归并排序


#### 快速排序


### 非比较类排序


#### 桶排序


#### 计数排序


#### 基数排序


#### 堆排序
